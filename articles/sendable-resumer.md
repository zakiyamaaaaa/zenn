---
title: "Swift復帰勢のための@Sendable"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift,Swift concurrency,Sendable]
published: false
---

## はじめに
Swift6に対応するには、`Sendable`プロトコルに対応することが必須となります。
この記事は、Swift6未満ではどのように対応していたのかも踏まえて、Swiftでの開発を最近復帰した方々でも`Sendable`を理解するための記事となります。
もちろん、新規の方も`Sendable`対応の流れが理解できます。

## `Sendable`とは
公式から`Sendable`についての概要を引用します。
https://developer.apple.com/documentation/swift/sendable
> データ競合のリスクを招かずに、任意の同時コンテキスト間で値を共有できるスレッドセーフな型（protocol）

Swift6では並行処理が厳格になり、並行処理（async関数やTask.detached など）でクロージャを渡す場合に `@Sendable`（以降`Sendable`と省略） が必要になるケースが増えました。
`Sendable`はSwift5.5から導入された非同期処理システム（Swift concurrency）の一部で、Swift6からは並行処理のコンパイルチェックが厳格になり、`Sendable`に準拠していないとエラーが出るようになりました。
そのため、`Sendable`を理解するにはSwift concurrencyでの重要な要素（async/await, Task, actor・・・）と絡んでくることが多いため、それらについてざっくりとでも知っていたほうがいいです。
それらの説明はほかの記事を参考にしていただき、この記事ではそうした他の要素を知らずとも`Sendable`を理解できるように焦点を当てて説明していきます。

## 役割
`Sendable`の役割としては、大きく3つになります
1. クロージャがスレッドセーフであることを保証
2. クロージャのキャプチャ（selfなどの参照）が安全であることの保証
3. 非同期タスクでのデータ競合を防ぐ

具体的な説明は後ほど行うとして、まずはこれらを以前（Swift6未満）はどのような形で対応していたのかを説明していきます。

## Swift6未満のスレッドセーフの方法
並行処理のスレッドセーフを保証することは、アプリケーション開発にとって重要な部分です。
Swift6未満では、どのように対応していたかというと`DispatchQueue（GCD）`,`NSLock`をもちいて対応していました。

### DispatchQueueでのスレッドセーフ対応
Swift６未満では次のようなコードで、並行処理を行うときにDispatchQueue（GCD）を使用して手動でスレッド管理を行っていました。

```swift
class Counter {
    private var value = 0
    private let queue = DispatchQueue(label: "com.example.counter", attributes: .concurrent)

    func increment() {
        queue.async(flags: .barrier) {
            self.value += 1
        }
    }

    func getValue() -> Int {
        queue.sync {
            return self.value
        }
    }
}
```
ポイントとしては、
* .concurrentを用いて並行処理を許可しつつ、書き込み（increment())のときに.barrierで排他制御
* getValueでは.syncを実行し、一貫性を確保

### NSLock
GCDの.barrierを使う代わりにNSLockでロック機構を使うこともできます。

```swift
class Counter {
    private var value = 0
    private let lock = NSLock()

    func increment() {
        lock.lock()
        value += 1
        lock.unlock()
    }

    func getValue() -> Int {
        lock.lock()
        let result = value
        lock.unlock()
        return result
    }
}
```
lock.lock() / lock.unlock() を使って、複数のスレッドが同時に value を変更しないように制御

### actor(swift5.5~)
Swift5.5以降ではactorを使うことで、スレッドセーフを保証できるようになりました。
```swift
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        return value
    }
}
```
* actorはデフォルトでスレッドセーフとなる
* awaitを使わないとactorのメソッドが呼び出せないため、必然的にスレッドセーフなコードとなる

```swift
let counter = Counter()
await counter.increment()
let currentValue = await counter.getValue()
```

### @escapingクロージャのCapture Listを使う
クロージャ内の循環参照を防ぐために、以前Swiftをやったことある方は見慣れていると思いますが、[weak self]や[unowned self]を使って、循環参照を防ぐのが一般的でした。

```swift
class Example {
    var value = 0

    func startTask() {
        DispatchQueue.global().async { [weak self] in
            self?.value += 1
        }
    }
}
```
弱参照とすることで、selfが開放されたあとにメモリリークを防ぎます。

このように様々な方法で、スレッドセーフを防ぐ方法がありましたが、これらは手動での管理であり、コンパイラのチェックが効かなかったりするため、とても不安定なものでした。
Swift6以降の@`Sendable`はこうした手動によるスレッドセーフの管理の問題を解決することが期待されます。

それでは具体的な`Sendable`の使い方や必要となるケースを学んでいきましょう。

## `Sendable`が必要となるケース
### Task.detachedを使う場合
```swift
Task.detached(priority: .userInitiated) { @Sendable in
    print("並行処理中")
}
```
Task.detachedは新しいスレッドを作成し、クロージャが平行実行されるため、安全性を保証する`Sendable`が必要となります。

### actorでクロージャを渡す場合
```swift
actor Counter {
    var value = 0
    
    func increment() {
        Task { @Sendable in
            value += 1  // ❌ エラー！ `value` は ``Sendable`` ではない
        }
    }
}
```
* actorのプロパティはスレッドセーフではないため、`@Sendable`なクロージャ内で直接アクセスできない
このような場合、下記のようにawaitをつけて対応できます。

```swift
actor Counter {
    var value = 0
    
    func increment() {
        Task { @Sendable in
            await self.incrementValue()
        }
    }
    
    func incrementValue() {
        value += 1
    }
}
```

## `Sendable`が必要となる具体例
```swift
func process(completion: @escaping () -> Void) {
    Task {
        completion()  // ❌ エラー: クロージャは `@`Sendable`` である必要がある
    }
}
```
completionが`Sendable`ではないため、Task内で安全に扱うことができないため、エラーとなります。

そのため、簡易的な修正としては、@escapingと同じく@`Sendable`を付与して回避できます。

```swift
func process(completion: @escaping @Sendable () -> Void) {
    Task {
        completion()  // ✅ エラー解消
    }
}
```
これによって、クロージャ内が`Sendable`であることを保証し、エラーを解決することができます。

## `Sendable`の制約
### 1. キャプチャされる変数は`Sendable`でなければならない
```swift
var counter = 0
Task { @Sendable in
    counter += 1  // ❌ エラー：`counter` は `Sendable` ではない
}
```
この場合変数counterが`Sendable`に準拠していないため、エラーとなります。
そのため、`Sendable`となるクラス・変数は`Sendable`としてください。

### 2. selfをキャプチャする場合はweakにする
```swift
class Example {
    var value = 0

    func startTask() {
        Task { @Sendable [weak self] in
            self?.value += 1  // ✅ 安全にキャプチャ
        }
    }
}
```

## おわり
以上が今回の記事となります。
普段業務ではFlutterでの開発を行っているのですが、趣味の開発でちょっとVisionOSやサードパーティライブラリを使うときに、少しハマってしまい、`Sendable`の知識が必要なところがありました。
これまでは実装者まかせであったスレッドセーフな処理が、コンパイラチェックを通して可能となったため、開発の安定性が向上するのではと期待しています。
結局慣れの問題だと思いますが、Swift6での開発では必須の知識だと思うので、ぜひこちらの記事が参考になれば幸いです。

## 参考にした記事とか
https://www.swift.org/migration/documentation/migrationguide/

https://developer.apple.com/documentation/swift/sendable


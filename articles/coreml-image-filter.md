---
title: "【SwiftUI】CoreML, Visionを使って画像加工アプリを作る"
emoji: "🧑‍🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [SwiftUI, Swift, CoreML, Vision, AI, 機械学習]
published: true
---

# はじめに
iOSの機械学習である`CoreML`を使って画像の加工をやってみたいと思います。
`CoreML`の概要について、[Apple公式ページ](https://developer.apple.com/jp/documentation/coreml/)から引用します。

> Core MLを使って、機械言語モデルをアプリに統合します。Core MLはすべてのモデルを統一された方法で扱えます。アプリは、Core ML APIとユーザーデータを使用して、予測の作成、モデルのトレーニングや微調整をすべてユーザーのデバイス上で行います。

最近では、デバイス上でモデルを作成できたり、そのスピードが向上したりしており、画像生成とかも可能になっています。
しかし、今回は手軽に画像加工を行うために、すでにあるモデルを使用してアプリを実装していきます。

また、ここでは`Visionフレームワーク`（後述ではVision）を使っています。`Vision`は、画像認識系の機械学習フレームワークで画像や動画に対応します。[Apple公式ページ](https://developer.apple.com/documentation/vision/)

`Vision`のほうもOSアップデートに伴い、様々な機能が追加・アップデートされており、リアルタイムの深度測定や画像分類などが可能となります。

対応バージョンについてですが、`CoreML`,`Vision`ともにiOS 11.0~なので、2025年現在では、ほぼすべてのアプリで対応可能となります。

次章から実装についての具体的な説明に入りますが、その前に実行環境を参考的に示します。Xcodeが動く環境であれば、だいたい大丈夫だと思います。

* macOS: Sequoia 15.2
* Xcode: 16.2
* Swift: 5

## 実装
### 流れ
まず、今回つくるアプリの流れを説明します。
1. ローカルの画像が入ってる状態でアプリが起動
2. ボタンを押すと、対象の画像がモデルによって加工処理
3. 加工が終わったら、加工後の画像を表示

という簡単な流れになります。
それでは早速作り始めたいところですが、まず初めに今回使用するCoreMLのモデルをダウンロードしていきます。

### Modelインポート
今回はこちらのソースからCoreMLモデルをダウンロードしていきます。なおダウンロードは自己判断とします。

https://github.com/john-rocky/CoreML-Models

今回行うタスクはImage2Image（画像から画像）となるので、Image2Imageの中から適当に選んでみてください。
一応自分が動かしたのは、[animeGANv2_paprika](https://github.com/john-rocky/CoreML-Models?tab=readme-ov-file#animeGANv2_paprika)となります。
指定のGoogle Driveからダウンロードしてきた`.mlmodel`のファイルをプロジェクトのフォルダに移動します。

![](/images/coreml-image-filter/image1.png)

これだけで、モデルのインポートは終わりです。
インポートして、Xcodeから該当モデルを選択するとこんな感じで、モデルの詳細が見ることができます。ここらへんの機能とかは、この記事では扱いません。

![](/images/coreml-image-filter/image2.png)

### Visionでモデルを扱う
それでは、先ほどいれたモデルを使っていきます。
モデルの参照はこんな感じです。

```swift
let model = try? VNCoreMLModel(for: animeganPaprika(configuration: .init()).model)
```

ここの`animeganPaprika`がモデルのファイル名なのですが、Xcodeのほうで自動的に認識して、補完が効いて入力できるようになっています。

このモデルを使って、モデルがどのようなタスクかを予測していきます。

```swift
let request = VNCoreMLRequest(model: model) { request, error in ...
```

返却値としては、画像分類の場合は、`VNClassificationObservation`。今回のimage2imageの場合は、`VNPixelBufferObservations`。それ以外はの場合は`VNRecognizedObjectObservation`または`VNCoreMLFeatureValueObservation`となります。

画像を加工する処理のコードとしては、次のようになります。
`isProcessing`は加工中かどうかのフラグです。

```swift
private func processImage(_ image: UIImage) {
        isProcessing = true
        guard let model = try? VNCoreMLModel(for: animeganPaprika(configuration: .init()).model),
              let ciImage = CIImage(image: image) else {
            print("Failed to load model or image")
            isProcessing = false
            return
        }

        let request = VNCoreMLRequest(model: model) { request, error in
            DispatchQueue.main.async {
                if let results = request.results as? [VNPixelBufferObservation],
                   let firstResult = results.first {
                    self.processedImage = UIImage(pixelBuffer: firstResult.pixelBuffer, targetSize: image.size)
                } else {
                    print("No results found: \(String(describing: error))")
                }
                self.isProcessing = false
            }
        }

        let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
        DispatchQueue.global(qos: .userInitiated).async {
            do {
                try handler.perform([request])
            } catch {
                print("Error performing request: \(error)")
                self.isProcessing = false
            }
        }
    }
```

### コード
全体のコードは次のようになります。`originalImage`の参照先は適宜変更してください。

```swift
import SwiftUI
import CoreML
import Vision

struct ContentView: View {
    @State private var originalImage = UIImage(resource: .sample)
    @State private var processedImage: UIImage?
    @State private var isProcessing: Bool = false

    var body: some View {
        VStack {
            
                Image(uiImage: originalImage)
                    .resizable()
                    .scaledToFit()
                    .frame(height: 200)
                    .padding()

            if let processedImage = processedImage {
                Image(uiImage: processedImage)
                    .resizable()
                    .scaledToFit()
                    .frame(height: 200)
                    .padding()
            } else if isProcessing {
                ProgressView("Processing Image...")
            }

            Button("Process Image") {
                    processImage(originalImage)
            }
            .padding()
        }
    }

    private func processImage(_ image: UIImage) {
        isProcessing = true
        guard let model = try? VNCoreMLModel(for: animeganPaprika(configuration: .init()).model),
              let ciImage = CIImage(image: image) else {
            print("Failed to load model or image")
            isProcessing = false
            return
        }

        let request = VNCoreMLRequest(model: model) { request, error in
            DispatchQueue.main.async {
                if let results = request.results as? [VNPixelBufferObservation],
                   let firstResult = results.first {
                    self.processedImage = UIImage(pixelBuffer: firstResult.pixelBuffer, targetSize: image.size)
                } else {
                    print("No results found: \(String(describing: error))")
                }
                self.isProcessing = false
            }
        }

        let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
        DispatchQueue.global(qos: .userInitiated).async {
            do {
                try handler.perform([request])
            } catch {
                print("Error performing request: \(error)")
                self.isProcessing = false
            }
        }
    }
}

extension UIImage {
    convenience init?(pixelBuffer: CVPixelBuffer, targetSize: CGSize) {
        let ciImage = CIImage(cvPixelBuffer: pixelBuffer)
        let context = CIContext()
        guard let cgImage = context.createCGImage(ciImage, from: ciImage.extent) else {
            return nil
        }
        let renderer = UIGraphicsImageRenderer(size: targetSize)
        let resizedImage = renderer.image { _ in
            UIImage(cgImage: cgImage).draw(in: CGRect(origin: .zero, size: targetSize))
        }
        self.init(cgImage: resizedImage.cgImage!)
    }
}

#Preview {
    ContentView()
}
```

実行すると、次のようになります。

![](/images/coreml-image-filter/movie1.gif)

# おわりに
たぶん、あまり業務的に使われることがない画像加工なんですが、CoreMLを知るためには良い教材かな、と思いました。そして、実際に動かしてみてその処理速度がほぼ一瞬だったのに驚きました。生成AIが最近は流行りですが、生成AIではない機械学習系技術も面白いと思います。
ただ、一つ課題としては、モデルによる加工の強度をアプリ側で調整する方法が見つけられませんでした。AIに聞いたところ、元画像とマスクして行う方法を提示されてやってみたのですが、うまくできず。ここらへんを次の課題として挑戦してみます。
ご覧いただきありがとうございました。

https://x.com/yamazaking0

# 参考にした記事
https://developer.apple.com/jp/documentation/coreml/

https://developer.apple.com/documentation/vision/

https://github.com/john-rocky/CoreML-Models




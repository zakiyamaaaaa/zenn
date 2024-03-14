---
title: "【Swift】CoreML VisionKitで手書き数字を認識してみる"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Swift, swiftui, coreml, visionkit, AI]
published: false
---

# はじめに
近年、人工知能(AI)技術の進化は目覚ましいものがあります。AppleはSiriをはじめ、従来から機械学習やAIの分野に注力してきました。今回は、オンデバイス上で実行可能なCoreMLとVisionKitを用いて、iPhoneでのテキスト認識の実装を試みたいと思います。

## Core ML
iOS 13~の機能でデバイス上で動作します。
*"Core MLは、Appleのハードウェアを活用し、メモリ占有量と電力消費量を最小限に抑えながら、幅広い種類のモデルがオンデバイスでパフォーマンスを発揮できるよう最適化されています。"* [Apple公式](https://developer.apple.com/jp/machine-learning/core-ml/)
また、PyTorchなどのフレームワークで作成したモデルでも使えるようです。以前はデバイスの性能不足で一般向けには満足したものは作りづらい印象でしたが、デバイスの性能は年々向上しており、Core ML自体もアップデートされているので、今後利用シーンは増えていくと予想されます。

まだ詳しく使ってはないのですが、Xcode上でモデルの挙動やパフォーマンスを検証できます。
![](/images/visionkit-coreml-number-recognition/image0.png)

## VisionKit
iOS 13~の機能でこちらもデバイス上で動作します。Core MLとの違いは画像認識に特化した機能ということです。
*"Identify and extract information in the environment using the device’s camera, or in images that your app displays.(デバイスのカメラを使用して環境内の情報、またはアプリが表示する画像内の情報を識別して抽出します。)"* [Apple公式](https://developer.apple.com/documentation/visionkit)

カメラ上のテキストやバーコードなどをほぼ遅延なく認識できたりします。
結構クレジットカードの番号入力省略とかの機能を既存アプリでも使っていたりしているので、こちらのほうがプロダクトに採用しているところが多い印象です。

WWDC2023では、画像のなかの人物や物体を自動的に切り出したりすることが大きな話題となりました。

https://developer.apple.com/videos/play/wwdc2023/10048/

今後はVisionOSとの組み合わせなど、視覚分野はAIでもHotな領域になる可能性が大いにあります。

# 実装の中身
それでは、今回の実装について説明していきます。

### 主な使用技術
- Core ML
- VisionKit
- PencilKit

### 動作環境
- Xcode 15.0
- iOS 17.0
- Swift 5.0

## Pencil Kit
今回手書き入力のときに共通して使用したのが`Pencil Kit`です
これによって、短いコードで簡単に手書き入力が可能となりました。

https://developer.apple.com/documentation/pencilkit

```swift
struct CanvasView: UIViewRepresentable {
    let canvas: PKCanvasView
    
    func makeUIView(context: Context) -> some UIView {
        canvas.backgroundColor = .white
        canvas.drawingPolicy = .anyInput
        let toolPicker = PKToolPicker()
        toolPicker.addObserver(canvas)
        toolPicker.setVisible(true, forFirstResponder: canvas)
        canvas.becomeFirstResponder()
        canvas.tool = PKInkingTool(.pen, color: .black, width: 30)
        return canvas
    }
    
    func updateUIView(_ uiView: UIViewType, context: Context) {}
    
    func reset() {
        canvas.drawing = PKDrawing()
    }
}
```

:::message
ちなみにここで`canvas.tool = PKInkingTool(.pen, color: .black, width: 30)`として幅を太めに指定してるんですが、当初デフォルトでやっていると１としか認識されなかったためです。おそらく細すぎて、うまく認識が反映されなかったと思われます。
:::

## Core ML
今回使用するモデルは、Apple公式が配布している[Core MLモデルのページ](https://developer.apple.com/jp/machine-learning/models/#image)で使いたいモデルをダウンロードします。
数字を認識したいため、MNIST（線画識別）のモデルを使います。

このダウンロードしたMNISTClassifier.modelをアプリフォルダの中にコピーします。

![](/images/visionkit-coreml-number-recognition/image1.png =300x)

このモデルを選択すると、↓のようにモデルの情報を見ることが出来ます。

![](/images/visionkit-coreml-number-recognition/image2.png)

Previewでは実際に画像をドラッグ＆ドロップして、精度を見ることが出来ます。
（あれ、5:信頼度100%となっている・・・・）

![](/images/visionkit-coreml-number-recognition/image3.png)

### 画像認識部分の実装
今回のケースでは、
1. モデルを定義
2. リクエストを作成
3. 使用する画像の取得
4. ハンドラーからリクエストを実行
という流れになります。
Core MLとVisionKitの違いとしてはVisionKitの場合はリクエストのところにすでに標準で用意された画像認識用等のAPIがあるので、モデルを定義する必要はありません。

### 1.モデルを定義
`VNCoreMLModel`のあとに使用するモデル名でモデルを指定します。

```swift
let model = try! VNCoreMLModel(for: MNISTClassifier.init(configuration: .init()).model)
```

### 2.リクエストを作成

```swift
let request = VNCoreMLRequest(model: model) { request, error in
            guard let results = request.results as? [VNClassificationObservation] else {return}
            recognizedNumber = results.first?.identifier ?? ""
        }
```

### 3.使用する画像の取得
Pencil KitのAPIには描画領域から画像を取得できる便利なAPIがあるので、こちらを利用します。

```swift
let inputImage = canvasView.canvas.drawing.image(from: canvasView.canvas.bounds, scale: 1)
```

### 4.ハンドラーからリクエストを実行
さきほど取得した画像をcgImageに変換したものをハンドラーに登録して、リクエストを実行します。

```swift
    let handler = VNImageRequestHandler(cgImage: inputImage.cgImage!)
    do {
        try handler.perform([request])
    } catch {
        print(error.localizedDescription)
    }
```

これが一連の実行フローです。
関数としてまとめると次のようになります。

```swift
    func recognize() {
        let model = try! VNCoreMLModel(for: MNISTClassifier.init(configuration: .init()).model)
        let request = VNCoreMLRequest(model: model) { request, error in
            guard let results = request.results as? [VNClassificationObservation] else {return}
            recognizedNumber = results.first?.identifier ?? ""
        }
        let inputImage = canvasView.canvas.drawing.image(from: canvasView.canvas.bounds, scale: 1)
        let handler = VNImageRequestHandler(cgImage: inputImage.cgImage!)
        do {
            try handler.perform([request])
        } catch {
            print(error.localizedDescription)
        }
    }
```
### 実行

UI部分を簡単にまとめます。

```swift
    @State private var recognizedNumber = ""
    let canvasView = CanvasView(canvas: PKCanvasView())

    var body: some View {
        VStack {
            Text(recognizedNumber)
            VStack {
                canvasView
                    .frame(width: 150, height: 150)
            }
            Button(action: {
                recognize()
            }, label: {
                Text("回答する")
            })
            Button(action: {
                canvasView.reset()
            }, label: {
                Text("Reset")
            })
        }
        .padding()
    }
```

それでは実機で実行します。

:::message
動作確認は実機で行う必要があります。
当初シミュレーターで行おうとしましたがエラーが出て、`usesCPUOnly = true`にするとシミュレーターでも動作するとありましたが、iOS 17.0からはdeprecatedとなっています。（代替方法はしめされていません。）

https://developer.apple.com/documentation/vision/vnrequest/2923480-usescpuonly
:::

![](/images/visionkit-coreml-number-recognition/coreml_sample.gif)

うーん、最初は良さそうだったんですが、精度はまだまだなようです。
これ以外にも７を1と認識したりすることが結構多かったです。今回使用したモデルは一桁の数字を学習させただけのモデルであるため、画像処理のところでもうひと工夫必要なのかもしれません（画像サイズの正規化やピクセル処理など）。

それでは次のVisionKitを利用した方法をやってみます。

## VisionKit
VisionKitの場合は、`import Vision`と書くだけで使えます。便利ですね。

### 画像認識部分の実装
Core MLのところでも言及したように、VisionKitはモデルを定義する必要はなく、リクエストのところで、いろいろな種類のリクエストを用途に応じて使うだけで大丈夫です。

関数としてまとめたもの

```swift
    func recognize() {
        let request = VNRecognizeTextRequest { request, error in
            if let results = request.results as? [VNRecognizedTextObservation] {
                let recognizedStrings = results.compactMap{ observation in
                    let str = observation.topCandidates(1).compactMap { text in
                        text.string
                    }
                    return str
                }.joined()
                inputText = recognizedStrings.joined()
            }
        }
        request.recognitionLevel = .fast
        let inputImage = canvasView.canvas.drawing.image(from: canvasView.canvas.bounds, scale: 1)
        let imageRequestHandler = VNImageRequestHandler(cgImage: inputImage.cgImage!)
        DispatchQueue.main.async {
            do {
                try imageRequestHandler.perform([request])
            } catch {
                print(error)
            }
        }
    }

```

### 実行

UI部分は次のようになってます。

```swift
@State var recognizedText = ""

    var body: some View {
        VStack {
            Text(recognizedText)
                .font(.system(size: 30))
            canvasView
                .frame(height: 200)
            Button(action: {
                recognize()
            }, label: {
                Text("Recognize")
            })
            Button(action: {
                canvasView.reset()
            }, label: {
                Text("Reset")
            })
        }
        .background(Color.green)
        .padding()
    }
```

それでは実行してみましょう。※Previewでも確認できます。

多分何も認識されない結果となったと思います。
どうやら画像に白背景を加えないと駄目らしいです。

https://note.com/sab_swiftlin/n/n9f1df57281b4

こちらの記事にあるようなやり方ではなく、ImageRenderを使って、Viewをそのまま画像として取り込む方法を使ってやってみました。

```swift
    private var outputView: some View {
        ZStack {
            if let inputImage = inputImage {
                Image(uiImage: inputImage)
            }
        }.background(.white)
    }

    @MainActor func recognize() {
        // 省略
        ・
        ・
        let canvasImage = canvasView.canvas.drawing.image(from: canvasView.canvas.bounds, scale: 1)
        inputImage = canvasImage
        let requestImage = ImageRenderer(content: outputView)
        let imageRequestHandler = VNImageRequestHandler(cgImage: requestImage.cgImage!)
        // 省略
    }

```

それでは実行してみます。

![](/images/visionkit-coreml-number-recognition/visionkit_sample.gif)

そこまで精度は悪くないまでも、数字以外のテキストへ認識されてしまったりしますね。数字の１についても、lやIと似ているため、一文字などコンテキストがない状態では、正しく認識するのは難しそうです。
ここらへんは認識候補から一番信頼度の高い数字のみ取り出す、という方法をとるなどすればもっとうまくいきそうです。

## まとめ
当初の目的だった手書きからの数字の認識というのは、現在のモデルでは精度としてあまり実用に耐えれるレベルではないな、と思いました。しかし、AIや機械学習の分野の成長速度は凄まじいので、おそらくそう遠くない未来にはここらへんが解決されていると思います。
お読みいただきありがとうございました。

## 参考にさせていただいた記事とか
https://developer.apple.com/jp/machine-learning/core-ml/

https://developer.apple.com/documentation/visionkit

https://note.com/sab_swiftlin/n/n9f1df57281b4
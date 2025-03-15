---
title: "Copilot VisionとClaude codeでiOSアプリを新規作成して比較"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Claude, Copilot, iOS, Xcode]
published: false
---

## はじめに
本記事の内容は、最近発表されたCopilot VisionとClaude codeを利用して、簡単なTODOアプリを作成する過程、そして実際にかかった料金（Claude code）について書きました。
これまでAIを使ったコーディングについては、VSCodeやCursorが先行していく形でしたが、ついにXcodeでも[GitHub公式からXcodeのアシスタントツール](https://github.com/github/CopilotForXcode)が発表され、コーディングアシストができるようになりました。
([サードパーティ製のCopilotForXcode](https://github.com/intitni/CopilotForXcode)が先行してリリースされて、使いましたが使用感はいまいちでした。)

また、最近ではChatGPTアプリからXcodeのファイルを参照できるCopilot Visionという機能もリリースされました。
一方、Claude codeはAnthropicが提供するコーディングエージェントで、Sonnetモデルを使用しています。

iOSアプリの場合、Xcodeを使うのですが、今回はCopilot VisionとClaude codeを使って、同じプロンプトを用いてTODOアプリを作成し、その結果を比較してみました。

:::message
なお、AIの料金体系や機能については、執筆時点（2025年3月15日）の情報であり、変更される可能性が高いことにご留意ください。
めまぐるしくAIサービスが発展しているため、1年後には撤廃されたりしている可能性もあります。
:::

## Copilot Vision
この機能を使うには、ChatGPTアプリを1.2025.057 以降に更新する必要があります。

:::message
自分の場合、ChatGPTアプリのcheck for updateを選択しても、更新が来なかったので、公式ページからインストールをしました。
また、今回使用したChatGPTのモデルは4.5になります。
:::

この状態から指示をだします。

![](/images/copilot-vision-claude-code-ios/image1.png)

ChatGPTアプリを開くと、このようにXcodeのアイコンが出てきます。
Xcodeアイコンが出ない場合は、チャット左下の画像で青いアイコンを選択して、Xcodeを選択してください。

![](/images/copilot-vision-claude-code-ios/image2.png)

こちらを選択して、読み込ませるファイル先を指定できます。

それでは簡単にプロンプトを出していきます。

```prompt:prompt
TODOアプリを作成してください
```

レスポンスは次のようでした。

:::details レスポンス
```swift:ContentView.swift
//
//  ContentView.swift
//  ChatGPT-Vision
//

import SwiftUI

struct ContentView: View {
    @State private var todos: [String] = []
    @State private var newTodo = ""

    var body: some View {
        VStack {
            HStack {
                TextField("New Todo", text: $newTodo)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .padding()

                Button(action: addTodo) {
                    Image(systemName: "plus.circle.fill")
                        .font(.title)
                }
                .padding()
                .disabled(newTodo.isEmpty)
            }

            List {
                ForEach(todos, id: \.self) { todo in
                    Text(todo)
                }
                .onDelete(perform: deleteTodo)
            }
        }
    }

    private func addTodo() {
        guard !newTodo.isEmpty else { return }
        todos.append(newTodo)
        newTodo = ""
    }

    private func deleteTodo(at offsets: IndexSet) {
        todos.remove(atOffsets: offsets)
    }
}

#Preview {
    ContentView()
}
```

TODOアプリ用にContentView.swiftを編集しました。
	•	新しいTODOを追加する入力欄とボタンを追加しました。
	•	リスト表示と削除機能を追加しました。

これでシンプルなTODOアプリが完成します！
```
:::

こちらコードを入れ替えた場合、次のようなプレビューになりました。

![](/images/copilot-vision-claude-code-ios/image3.png)

コードを見た限り、一応見た目はできてそうですが、一時的な保存となり、アプリをタスクキルした場合、初期化されてしまいます。

### 料金
Copilot Visionの料金なんですが、調べてもちょっと出てこなかったです。GPT4oから使えるということなので、Freeプランでも使えるかもしれません。

## Claude Code
Claude CodeはClaude 3.7 Sonnetをモデル使用したコーディングエージェントです。
使用するには、Anthropicのアカウントが必要です。
APIの使用には料金がかかりますが、クレジットで事前にチャージして、そこから支払われるので、予想外の支払いを抑えることができます。

https://console.anthropic.com/dashboard

それでは、Claude Codeを使っていきます。

インストールについては、こちらの記事を参考にしました。
https://zenn.dev/schroneko/articles/a4496472b76477

無事インストールできたら、Claude Code用にXcodeプロジェクトを新規作成します。
ディレクトリに移動して、claudeを起動します。

```bash
claude
```

次に、Copilot Visionと同じようにプロンプトを出します。

```prompt:prompt
TODOアプリを作成してください
```

ファイルを自動的に生成・変更してくれます。

:::details レスポンス

> TODOアプリを作成してください

⏺ SwiftUIでTODOアプリを作成します。まずは必要なファイルを作成していきます。

⏺ Call(Create TODO app model files)…
  ⎿  Updated Claude-app/Models/TodoItem.swift with 15 additions
      1  //
      2  //  TodoItem.swift
      3  //  Claude-app
      4  //
      5  //  Created by Claude on 2025/03/15.
      6  //
      7  
      8  import Foundation
      9  
     10  struct TodoItem: Identifiable, Codable {
     11      var id = UUID()
     12      var title: String
     13      var isCompleted: Bool = false
     14      var createdAt: Date = Date()
     15  }
  ⎿  Updated Claude-app/ViewModels/TodoListViewModel.swift with 50 additions
      1  //
      2  //  TodoListViewModel.swift
      3  //  Claude-app
      4  //
      5  //  Created by Claude on 2025/03/15.
      6  //
      7  
      8  import Foundation
      9  import SwiftUI
     10  
     11  class TodoListViewModel: ObservableObject {
     12      @Published var todos: [TodoItem] = []
     13      
     14      private let todosKey = "todos"
     15      
     16      init() {
     17          loadTodos()
     18      }
     19      
     20      func addTodo(title: String) {
     21          let newTodo = TodoItem(title: title)
     22          todos.append(newTodo)
     23          saveTodos()
     24      }
     25      
     26      func toggleCompletion(for todo: TodoItem) {
     27          if let index = todos.firstIndex(where: { $0.id == todo.id }) {
     28              todos[index].isCompleted.toggle()
     29              saveTodos()
     30          }
     31      }
     32      
     33      func deleteTodo(at indexSet: IndexSet) {
     34          todos.remove(atOffsets: indexSet)
     35          saveTodos()
     36      }
     37      
     38      private func saveTodos() {
     39          if let encoded = try? JSONEncoder().encode(todos) {
     40              UserDefaults.standard.set(encoded, forKey: todosKey)
     41          }
     42      }
     43      
     44      private func loadTodos() {
     45          if let todosData = UserDefaults.standard.data(forKey: todosKey),
     46             let savedTodos = try? JSONDecoder().decode([TodoItem].self, from: todosData) {
     47              todos = savedTodos
     48          }
     49      }
     50  }
  ⎿  Done (2 tool uses · 8.4s)

⏺ Call(Create TODO app view files)…
  ⎿  Updated Claude-app/Views/TodoListView.swift with 88 additions
      1  //
      2  //  TodoListView.swift
      3  //  Claude-app
      4  //
      5  //  Created by Claude on 2025/03/15.
      6  //
      7  
      8  import SwiftUI
      9  
     10  struct TodoListView: View {
     11      @StateObject private var viewModel = TodoListViewModel()
     12      @State private var newTodoTitle = ""
     13      
     14      var body: some View {
     15          NavigationView {
     16              VStack {
     17                  HStack {
     18                      TextField("新しいタスク", text: $newTodoTitle)
     19                          .textFieldStyle(RoundedBorderTextFieldStyle())
     20                      
     21                      Button(action: addTodo) {
     22                          Image(systemName: "plus.circle.fill")
     23                              .font(.title2)
     24                      }
     25                      .disabled(newTodoTitle.isEmpty)
     26                  }
     27                  .padding()
     28                  
     29                  List {
     30                      ForEach(viewModel.todos) { todo in
     31                          TodoItemView(todo: todo) { 
     32                              viewModel.toggleCompletion(for: todo)
     33                          }
     34                      }
     35                      .onDelete { indexSet in
     36                          viewModel.deleteTodo(at: indexSet)
     37                      }
     38                  }
     39              }
     40              .navigationTitle("TODOリスト")
     41              .toolbar {
     42                  EditButton()
     43              }
     44          }
     45      }
     46      
     47      private func addTodo() {
     48          if !newTodoTitle.isEmpty {
     49              viewModel.addTodo(title: newTodoTitle)
     50              newTodoTitle = ""
     51          }
     52      }
     53  }
     54  
     55  struct TodoItemView: View {
     56      let todo: TodoItem
     57      let toggleAction: () -> Void
     58      
     59      var body: some View {
     60          HStack {
     61              Button(action: toggleAction) {
     62                  Image(systemName: todo.isCompleted ? "checkmark.circle.fill" : "circle")
     63                      .foregroundColor(todo.isCompleted ? .green : .primary)
     64              }
     65              
     66              Text(todo.title)
     67                  .strikethrough(todo.isCompleted)
     68                  .foregroundColor(todo.isCompleted ? .gray : .primary)
     69              
     70              Spacer()
     71              
     72              Text(formattedDate)
     73                  .font(.caption)
     74                  .foregroundColor(.gray)
     75          }
     76      }
     77      
     78      private var formattedDate: String {
     79          let formatter = DateFormatter()
     80          formatter.dateStyle = .short
     81          formatter.timeStyle = .short
     82          return formatter.string(from: todo.createdAt)
     83      }
     84  }
     85  
     86  #Preview {
     87      TodoListView()
     88  }
  ⎿  Done (1 tool use · 32.3s)

⏺ Create(file_path: Claude-app/Claude_appApp.swift)…
  ⎿  Cannot create new file - file already exists.

⏺ Read(file_path: "Claude-app/Claude_appApp.swift")…
  ⎿  //
     //  Claude_appApp.swift
     //  Claude-app
     ... (+15 lines)

⏺ Update(file_path: Claude-app/Claude_appApp.swift)…
  ⎿  Updated Claude-app/Claude_appApp.swift with 1 addition and 1 removal
     11  struct Claude_appApp: App {
     12      var body: some Scene {
     13          WindowGroup {
     14              ContentView()
     14              TodoListView()
     15          }
     16      }
     17  }

⏺ Bash(mkdir -p /Users/shoichiyamazaki/Programming/swift/Claude-app/Claude-app/Models …
      /Users/shoichiyamazaki/Programming/swift/Claude-app/Claude-app/ViewModels
      /Users/shoichiyamazaki/Programming/swift/Claude-app/Claude-app/Views)
  ⎿  (No content)

⏺ TODOアプリを作成しました。以下の機能を持つアプリです：

  1. タスクの追加、削除、完了/未完了の切り替えができる
  2. UserDefaultsを使ってタスクを保存
  3. 日本語UIで「新しいタスク」の入力フォームと「TODOリスト」のタイトル

  アプリを実行するとTODOリスト画面が表示され、テキストフィールドに新しいタスクを入力し、プラスボタンをタ
  ップでタスクを追加できます。タスク項目の左側のチェックボタンで完了/未完了を切り替えられます。
:::

アプリの外観としては、このようになりCopilot Visionとほぼ同じとなりました。（同様のコードを参照している？）

![](/images/copilot-vision-claude-code-ios/image7.png)

結果的に次のような構成となりました。

![](/images/copilot-vision-claude-code-ios/image4.png)

どうやらMVVMとしているようです。
レスポンスにあるコードの説明でも、UserDefaultsを使ってタスクを保存していることがわかります。

### 料金
Claude Codeの料金は、APIの利用による従量課金です。
今回作ってみたアプリでの使用量は、0.17ドル（≒25円）でした。

## 比較
今回はCopilot VisionとClaude Codeを使って、同じプロンプトを用いてTODOアプリを作成しました。
どちらもSwiftUIを使用して、リスト表示と追加・削除機能を実装しています。
しかし、質としては圧倒的にClaude Codeが優勢でした。Copilot Visionは、一時的な保存であり、アプリをタスクキルした場合、初期化されてしまいます。一方、Claude CodeはUserDefaultsを使ってタスクを保存しているため、アプリを再起動してもデータが残っていました。

また、Copilot Visionについては、他のファイルを生成して考慮するなどできないため、そこもClaude Codeに比べて劣っていると感じました。ただ、そこらへんはプロンプトの内容を改良すれば、うまくいくのかもしれません。

Copilot Visionについては、Copilotに課金している必要かどうかはわかりませんでした。一応GitHub Copilotは、Freeのサブスクリプションでも使えるようでした。

| ツール | Copilot Vision | Claude Code |
| ---- | ---- | ---- |
| コードの質 | 低 | 中 |
| 料金 | 無料？ | API利用の従量課金 |
| 結果 | ![](/images/copilot-vision-claude-code-ios/image3.png =100x) | ![](/images/copilot-vision-claude-code-ios/image7.png =100x) |

## まとめ
iOSアプリ開発でもAIを使ったコーディング支援ツールが登場しました。
今回は最近話題のCopilot VisionとClaude Codeを使って、同じプロンプトを用いてTODOアプリを作成しました。
結果的にはClaude Codeのほうが優勢でしたが、プロンプトを改良すれば、Copilot Visionも使いやすくなるかもしれません。
なお、Claude Codeはもちろんですが、Copilot Visionのほうでも、コードを自動的に変更することが可能となるため、コピペがいらなくなりました。
また、今回の例では新規開発で使いましたが、実際には既存コードの修正や追加であったり、もっと複雑なアプリになることが多いと思います。なので、この記事では費用感と出力を参考程度にしていただければと思います。

## 参考資料
https://github.com/github/CopilotForXcode

---
title: "Riverpod Hooksを使ってAPIリクエストをキャッシュする"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# 趣旨
一般的なFlutterのアプリにおいて、Riverpodを使ってAPIリクエストから返ってくるデータを扱うことはよくあると思います。
次のコードはAPIリクエスト（poke API）を行い、返ってきたデータをListViewに表示するというよくある例の抜粋です。

```dart
@override
  Widget build(BuildContext context, WidgetRef ref) {
    final pokemons = ref.watch(restApiProvider).getPokemon(limit, offset);
    // some codes here
    return FutureBuilder<Pokemons>(
        future: pokemons,
        builder: (context, snapshot) {
            // Loading
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const CircularProgressIndicator();
            // Error
          if (snapshot.hasError) {
            return Text('Error: ${snapshot.error}');
            // Success
          return ListView.builder(
            itemCount: snaphost.hadData ? snapshot.data!.results.length : 0,
            itemBuilder: (context, index) {
                // some codes here
            });
            }
          }
        }
    );
  }
```

しかし、この場合APIリクエストが複数呼ばれてしまいます。


# 本記事で扱う技術・ツール
* Flutter
* useMemoized
* useFuture
* DevTools(Network)

バージョンは次のとおりです
* Flutter 3.19.6
* flutter_riverpod: 2.5.1
* hooks_riverpod 2.5.1
* flutter_hooks: 0.20.5
* DevTools: 2.28.5

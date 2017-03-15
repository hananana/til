# Vim8でUnityを超速開発する処方箋

この記事は以下の症状を訴えている方に効能があります。

- monodevelop遅い。起動時間のストレスで髪が抜ける。
- Xamarin遅い。補完に一万光年かかってるせいで胃酸がスプラッシュ。
- VisualStudio買えない。お金にゃい。うつだしのう。

## 主な効能

- 起動してコードを書くまでミリ秒です。
- 数秒で別プロジェクトの関数をコピペできます。
- .asset, .prefab, .unityファイルも編集できます。
- ファイル検索、grepもミリ秒です。
- 貴方の情熱次第でいくらでもIDEを上回る機能を追加できます。

## オペ開始

```sh
brew install python3
brew install vim --with-client-server --with-python3 --with-override-system-vi
```


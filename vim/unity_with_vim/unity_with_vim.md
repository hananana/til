# Vim8でUnity音速開発する処方箋

※この記事で言う音速とは第３者がモニタを見て何してるんだか理解不能の速度を叩き出すことを言います。

この記事は以下の症状を訴えている方に効能があります。

- monodevelop遅い。起動時間のストレスで髪が抜ける。
- Xamarin重い。俺のMacBookAirだと補完に１万年かかって婚期を逃しそう。
- VisualStudio買えない。お金にゃい。うつだしのう。

## 主な効能

- 起動してコードを書くまでミリ秒です。
- 数秒で別プロジェクトの関数をコピペできます。
- .asset, .prefab, .unityファイルもエディタからそのまま編集できます。
- ファイル検索、grepもミリ秒です。
- 旧世代のMacでも大丈夫。Unity自体は重いけどね！
- 無料で環境構築できます。
- 貴方の情熱次第でいくらでもIDEを上回る機能を追加できます。

## 術式開始

```sh
brew install python3
brew install vim --with-client-server --with-python3 --with-override-system-vi
```


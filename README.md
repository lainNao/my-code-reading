# my-code-reading

個人的なコードリーディング記録

## React

- [ui-neumorphism](/code-reading-of-ui-neumorphism.md)
- [chakra-ui](/code-reading-of-chakra-ui.md)
- [mui](/code-reading-of-mui.md)
- [mantine](/code-reading-of-mantine.md)

## other

- [markdownlint](/code-reading-of-markdownlint.md)
- [zenn-editor](/code-reading-of-zenn-editor.md)
- [mercari-shops-automation](/code-reading-of-mercari-shops-automation.md)

---

TODO

- Reactで作られたオープンソースアプリケーション
- 設計パターンだけでなく個々のコンポーネントの実装の参考事例
  - 特にMUIのDataGridなどのbodyとheaderが別のdivになっていてスクロールが同期するやつ
- storybook回りの色んなパターン
- react-hook-form回りの実装パターンのプラクティス
  - 例えば「TextFieldとラベルとエラーメッセージを混合したコンポーネント」を作る時、どう分割するか。
    - レイアウト用コンポーネントを流し込めるようにしたほうがきれいかも
    - そもそもそういう混合コンポーネントは作らない方が良かったりする？
    - TextFieldというatomがあったとして、それをRHFでwrapしたRHFTextFieldを作るパターンがあるけど、そういう感じがよいのか？
    - もっとやりやすい方法無いか？etc
  

# my-code-reading

個人的なコード/ドキュメントリーディング記録

## React

- [ui-neumorphism](/code-reading-of-ui-neumorphism.md)
- [chakra-ui](/code-reading-of-chakra-ui.md)
- [mui](/code-reading-of-mui.md)
- [mantine](/code-reading-of-mantine.md)

## markdown

- [markdownlint](/code-reading-of-markdownlint.md)
- [zenn-editor](/code-reading-of-zenn-editor.md)

## Design

- [mercari-shops-automation](/doc-reading-of-mercari-shops-automation.md)
- [designtokens-org-format](/doc-reading-of-designtokens-org-format.md)


---

TODO

- 過去に読んだから省略するorもう一度読んでもいいかもなもの
  - 一昨年辺りにチャット実装の参考にしようとして読んだもの
    - Rocket Chat
      - Meteorを使っていて結構ソースコードが独自的だった記憶がある。そもそもMeteorがwebsocket上でAPIのやり取りをするという珍しい形式だった。基本それに依存していてスケールしなさそうではあった。
    - Mattermost
      - bootstrapのreactラッパーを使っていた。ソース的には簡素だった気がする。規模に合わせたスケール別の環境構築のあれこれが公式サイトに詳しく書かれていて良かった
- Reactで作られたオープンソースアプリケーション
  - なんか色々あった気がする
- Reactのコンポーネントライブラリ
  - ant designとか、syncfusionとか、devextremeとか
- 設計パターンだけでなく個々のコンポーネントの実装の参考事例
  - 特にMUIのDataGridなどのbodyとheaderが別のdivになっていてスクロールが同期するやつ
- 全て同じ目次構造を持ってないとコミットできないようにする
  - markdown的に警告など出てたらコミットできないようにする
  - <https://github.com/lainNao/markdownlint-rule-trace-template-headers> を作ったのでCIに組み込む
- storybook回りの色んなパターン
- react-hook-form回りの実装パターンのプラクティス
  - 例えば「TextFieldとラベルとエラーメッセージを混合したコンポーネント」を作る時、どう分割するか。
    - レイアウト用コンポーネントを流し込めるようにしたほうがきれいかも
    - そもそもそういう混合コンポーネントは作らない方が良かったりする？
    - TextFieldというatomがあったとして、それをRHFでwrapしたRHFTextFieldを作るパターンがあるけど、そういう感じがよいのか？
    - もっとやりやすい方法無いか？etc
- 以下勉強
  - pnpm
  - デザインシステムのデザイントークンをいい感じに扱う方法
  - Formily <https://v2.formilyjs.org/> の存在を忘れてた
    - react-hook-formがデファクトスタンダードみたいになっているけどそういえばFormilyとか中国系のフォームライブラリも試したい
      - XRender内のFormRenderとかant-designに依存してはいるかもだけどどうなのか <https://xrender.fun/>
  - マイクロフロントエンド系ライブラリと、そのライブラリを使ってる団体が他に使ってるライブラリ
    - quinkun <https://qiankun.umijs.org/>
    - icestark <https://zenn.dev/mikana0918/articles/344861f49f7190>
    - single-spaなら名前知っている程度だった。
  - 久しぶりにマイクロフロントエンド系でググりたい
    - <https://blog.cybozu.io/entry/2022/12/21/110000>
  - なんかこういうの見つけた <https://ahooks.js.org/hooks/use-why-did-you-update>
  - react-useとahookの比較
  - CLI系ライブラリ
    - cac
    - inquirer.js

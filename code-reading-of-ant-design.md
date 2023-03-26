# 目次

- 結論
  - [参考になった箇所](#参考になった箇所)
  - [微妙だと思った箇所](#微妙だと思った箇所)
  - [他気になった箇所](#他気になった箇所)
- 色々
  - [使い方](#使い方)
  - [なぜ読もうとしたか](#なぜ読もうとしたか)
  - [URL](#url)
  - [使用ライブラリ](#使用ライブラリ)
  - [開発方法](#開発方法)
  - [リリース方法](#リリース方法)
  - [フォルダ構成やファイル構成](#フォルダ構成やファイル構成)
  - [自動テスト構成](#自動テスト構成)
  - [その他](#その他)

## 参考になった箇所

- 使用ライブラリの項目も参照。
- テーマ編集のページが頑張っていた。<https://ant-design.github.io/antd-token-previewer/editor>

## 微妙だと思った箇所

## 他気になった箇所

## 使い方

## なぜ読もうとしたか

## URL

## 使用ライブラリ

- 色やアイコンを別リポジトリで管理している。デザインシステムの構成要素の中でその2つが確かに始めやすいのかも。

  ```json
  "@ant-design/colors": "^7.0.0",
  "@ant-design/cssinjs": "^1.5.6",
  "@ant-design/icons": "^5.0.0",
  "@ant-design/react-slick": "~1.0.0",
  ```

  - `@ant-design/colors`にて、色を一つ色を与えればそれの明るさが異なる色も自動生成してくれるgenerateという関数を作って作成されていた。MUIも同じようなことやってるけどこういうのは良さそう。 <https://github.com/ant-design/ant-design-colors/blob/master/src/generate.ts>
- package.jsonを見たら、ものすごくrc-◯◯系のコンポーネントに依存しているのが分かる。しかもrc-◯◯はant design用のコンポーネントらしい。すごい。
  - <https://github.com/react-component>
- dnd-kit <https://github.com/clauderic/dnd-kit>
  - 良さそう。DnDは結構ライブラリあるけどこれは良い方な気がする。
- size-limit <https://github.com/ai/size-limit>
  - よいのかも。MUIも使っているっぽい。
- dekko <https://github.com/benjycui/dekko>
  - 指定フォルダに指定ファイルがあるかどうかのchecker的なもの。CIに使えるかもしれない。でも使う前に似たようなツール他に無いのか探してみたい。

## 開発方法

## リリース方法

## フォルダ構成やファイル構成

## 自動テスト構成

## その他

- いろんなdependenciesを見ていて思い出したけどFormily <https://v2.formilyjs.org/>ってどうなんだろう。
  - react-hook-formがデファクトスタンダードみたいになっているけど、こちらもそういえばあまり見てなかったから少し見てみたさはある。
- quinkunというのも見つけた。 <https://qiankun.umijs.org/>
  - マイクロフロントエンドのライブラリ。
    - 他にはicestarkとかもある
      - <https://zenn.dev/mikana0918/articles/344861f49f7190>
    - single-spaなら名前知っている程度だった。
    - マイクロフロントエンドで久しぶりにググったら前より明らかに情報増えていた。
      - <https://blog.cybozu.io/entry/2022/12/21/110000>
    - マイクロフロントエンドをググるのをTODOに加えたい
- なんかこういうの見つけた <https://ahooks.js.org/hooks/use-why-did-you-update>
  - 面白い
  - このahook <https://ahooks.js.org/hooks/use-request/index> とreact-useを比較してみたい。
- こういうサイト下部の利用団体リストを一個一個見ていくのも面白そう <https://actions-cool.github.io/issues-helper>

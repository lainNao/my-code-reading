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

- 環境周り
  - 良いドキュメントがある
    - mui/base、mui/materialなどのパッケージ構成の説明　<https://mui.com/material-ui/guides/understand-mui-packages/>
    - テストについて　<https://github.com/mui/material-ui/blob/HEAD/test/README.md>
    - VRTについて　<https://github.com/mui/material-ui/blob/a1b591aadb56cd829f5800c5c9987134f84644b9/test/regressions/README.md>
      - playwrightをスナップショットに使ってるらしい
    - E2Eについて　<https://github.com/mui/material-ui/blob/a1b591aadb56cd829f5800c5c9987134f84644b9/test/e2e/README.md>
  - @octokit/rest、chakraも使っていた
  - babel-plugin-react-remove-propertiesを使っていた。これはテスト時以外はdata-testidを消したい時に使えるやつ。babel以外にもほしい
  - dtslintがよさそう。型のテストに使う。型とデータをいくつか用意して、passし続けるかを確認するやつだと思う。ただ、ライブラリ用ではありそう
  - rollupというバンドラを使っていた
  - STYLUSはsxの実装に使っているもの？
  - lernaによるモノレポ
  - CIとかいろいろ。だいたいcircle CIで、一部github actions？
    - argoによるVRT: `ci/circleci: test_regression-1`
    - 複数ブラウザによる単体テスト: `test_browser-1`
    - JSDOM環境での単体テスト: `test_unit-1`
    - バンドルテスト: `test_bundling_バンドラ名`
    - 後は普通にlint, type, static, bundle sizeとかあテストしてる
    - CIで一時的なnetlifyのステージングサーバーにデプロイしたURLが貼ってある
  - dangerがいい。PRの文面や内容を見て、コメントやレビューをできるやつなはず。ユースケースは公式に記載のある通り色々ある。
- ソース周り
  - chakra-uiとかなり似ていて、おそらくmuiのソースを真似てchakraは作られている。
  - propsはコンポーネントと別ファイル（Alert.tsxの他にAlertProps.tsxが作られている等）
  - slotsという概念がある。これはvueのslotと似ており、「propsで入れられたらそのコンポーネントに置き換わり、入れられなかったらデフォルトのコンポーネントに置き換わる」という使われ方をしているっぽい。例えばコンポーネントの一部をspanじゃなくdivに変えるとかみたいななのかな…？
  - propsは()の中で展開せず、その下の行で展開している。なんとなくその方が良い気がしてきた。

## 微妙だと思った箇所

## 他気になった箇所

- storybookを使ってないけどどうやってクオリティを担保している？argosによるVRTはあるけれども

## 使い方

## なぜ読もうとしたか

## URL

<https://github.com/mui/material-ui>

## 使用ライブラリ

- argos-ci
  - VRTプラットホーム
- emotion
- @octokit/rest

## 開発方法

- <https://github.com/mui/material-ui/blob/master/CONTRIBUTING.md>
  1. リポジトリをフォーク
  2. フォークをローカル マシンに複製し、アップストリーム リモートを追加
      - `git clone https://github.com/<your username>/material-ui.git`
      - `cd material-ui`
      - `git remote add upstream https://github.com/mui/material-ui.git`
  3. masterローカルブランチをアップストリーム ブランチと同期
      - `git checkout master`
      - `git pull upstream master`
  4. yarn を使用して依存関係をインストールします (npm はサポートされていません)。
      - `yarn install`
  5. 新しいトピック ブランチを作成
      - `git checkout -b my-topic-branch`
  6. 変更を加え、コミットしてフォークにプッシュ
      - `git push -u origin HEAD`
  7. リポジトリに移動し、プル リクエストを作成

## リリース方法

## フォルダ構成やファイル構成

## 自動テスト構成

## その他

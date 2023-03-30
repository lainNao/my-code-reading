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

- ※以下は大まかに適当に読んだ結果なので間違っているかもしれない
  - デザイントークンの拡張子は`.tokens`か`.tokens.json`

  - デザイントークンは以下の形式で表現できる。※適当

    ```ts
    type SingleTokenType = "color" | "dimension " | "fontFamily" | "fontWeight" | "duration" | "cubicBezier" | "number";
    type CompositeTokenType = "border" | "shadow" | "box-shadow" | "strokeStyle" | "transition" | "gradient";

    type TokenType = SingleTokenType | CompositeTokenType

    type DesignTokenSummary = {
      "$type": TokenType,
      "$description": string,
      "$value": any,
      "$extensions" : any, //任意の拡張オブジェクト
    }

    type DesignToken = {
      "トークン名": DesignTokenSummary
    }

    type DesignTokenGroup = {
      "グループ名": {
        DesignTokenSummary & DesignToken[]
      }
    }
    ```

  - デザイントークンは、グループなものや複合なものもある。
- 関連ツールがあった
  - 変換ツール
    - <https://github.com/salesforce-ux/theo>
    - <https://amzn.github.io/style-dictionary/#/>
    - <https://specifyapp.com/>
    - <https://diez.org/>
  - ドキュメンテーションツール
    - <https://zeroheight.com/>
    - <https://backlight.dev/>
    - <https://specifyapp.com/>
    - <https://www.knapsack.cloud/>

## 微妙だと思った箇所

- なんで`$`をつけるのか。

## 他気になった箇所

## 使い方

json化して、後は各PJで使う。
より便利にするツールを使うかどうかはおまかせ。（変換ツール、ドキュメンテーションツール）

## なぜ読もうとしたか

簡単だったから。

## URL

<https://tr.designtokens.org/format/#use-cases>
<https://github.com/design-tokens/community-group>

## 使用ライブラリ

## 開発方法

## リリース方法

## フォルダ構成やファイル構成

## 自動テスト構成

## その他

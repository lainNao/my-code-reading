# 目次
- 結論
  - [参考になった箇所](#参考になった箇所)
  - [微妙だと思った箇所](#微妙だと思った箇所)
  - [他気になった箇所](#他気になった箇所)
- 色々
  - [なぜ読もうとしたか](#なぜ読もうとしたか)
  - [URL](#URL)
  - [使用ライブラリ](#使用ライブラリ)
  - [開発方法](#開発方法)
  - [リリース方法](#リリース方法)
  - [フォルダ構成/ファイル構成](#フォルダ構成/ファイル構成)
  - [自動テスト構成](#自動テスト構成)

# 内容

## 参考になった箇所
- 使ったこと無いプロジェクト管理技術が使われてる
  - changesetsでのバージョン管理
  - pnpmでのモノレポ。各コンポーネントが別でpackage.jsonなどを持っている
  - tsupでのtsからjsへのビルド
  - typedocでのドキュメント自動作成
  - @octokit/restでgithub操作
  - @commitlintでコミットルールの強制
  - edit-json-fileでのjson編集
  - find-packages, find-upによるパッケージ/ファイル検索
  - outdentによる余計な改行の削除
- ui-neumorphismのように、cloneElementをしている箇所があった（avatar-image.tsx）。ということは別にまあ使って良いのかなという印象になった。
- 特にreadonlyを毎回つけているわけではない。
- 毎回forwardRefをつけている
- ファイル名はケバブケース。ケースインセンシティブな環境対策かな
- propsのデフォルト値は、propsの段階でつけるのでなく、一旦const化した時につけている
  ```tsx
  // icon.tsx
  export const Icon = forwardRef<IconProps, "svg">((props, ref) => {
    const {
      as: element,
      viewBox,
      color = "currentColor",
      focusable = false,
      children,
      className,
      __css,
      ...rest
    } = props
    ...
  ```
- storiesのファイル、meta部分とstoryの部分でファイルを分けてるパターンがたまにあった。
  ```tsx
  // index.stories.tsx
  export * from "./menu.stories"  //こっちの方にstoryを書いていた

  export default {
    title: "Components / Overlay / Menu",
    decorators: [
      (story: Function) => (
        <chakra.div maxWidth="500px" mx="auto" mt="40px">
          {story()}
        </chakra.div>
      ),
    ],
  }
  ```
- コンポーネントは基本的にはスタイル用のProviderで囲っており、テーマで上書きできるようにしていた
  ```tsx
  //tag.tsx
  ...
  return (
    <TagStylesProvider value={styles}>
      <chakra.span ref={ref} {...ownProps} __css={containerStyles} />
    </TagStylesProvider>
  )
  ```
- テーマはEmotionThemeProviderに依存していた
  ```tsx
  export function ThemeProvider(props: ThemeProviderProps): JSX.Element {
    const { cssVarsRoot, theme, children } = props
    const computedTheme = useMemo(() => toCSSVar(theme), [theme])
    return (
      <EmotionThemeProvider theme={computedTheme}>
        <CSSVars root={cssVarsRoot} />
        {children}
      </EmotionThemeProvider>
    )
  }  
  ```
- CONTRIBUTING.mdがあって、コントリビュート方法が書いてあった。
- contextの使い方。そもそもあるべきProviderが祖先でセットされていなかったらエラーをthrowし、開発時に気づけるようにしている。型安全ではないけど安全？
  ```tsx
  export function useTheme<T extends object = Dict>() {
    const theme = useContext(
      ThemeContext as unknown as React.Context<T | undefined>,
    )
    if (!theme) {
      throw Error(
        "useTheme: `theme` is undefined. Seems you forgot to wrap your app in `<ChakraProvider />` or `<ThemeProvider />`",
      )
    }

    return theme as WithCSSVar<T>
  }
  ```


## 微妙だと思った箇所
- jestの単体テストは豊富であったが、MUIと違いVRTは無かった

## 他気になった箇所
- displayNameを毎回値を入れていたが、なんのメリットがあるのか分からなかった
- data-loaded的な属性を使ったりしているコンポーネントがある。data属性はなぜ使っているのか？useStateでも同じような値を二重管理していた。
- 他いろいろ

## URL
https://github.com/chakra-ui/chakra-ui/

## 使用ライブラリ
- 環境類
  - changesets
  - pnpm
  - tsup
  - typedoc
- 動作類
  - @emotion/react

## 開発方法
- https://github.com/chakra-ui/chakra-ui/blob/main/CONTRIBUTING.md
  - folkする
  - mainからブランチ作成
  - pnpm pkg <module> build、pnpm pkg <module> testで動作確認
  - pnpm changesetでchangelogを作成

## リリース方法
- changesets publish

## フォルダ構成/ファイル構成
- 普通のモノレポ

## 自動テスト構成
- VRT無し
- jest偏重

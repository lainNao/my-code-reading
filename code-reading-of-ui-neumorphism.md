# 目次
- 結論
  - [参考になった箇所](#参考になった箇所)
  - [微妙だと思った箇所](#微妙だと思った箇所)
- 色々
  - [なぜ読もうとしたか](#なぜ読もうとしたか)
  - [URL](#URL)
  - [使用ライブラリ](#使用ライブラリ)
  - [開発方法](#開発方法)
  - [リリース方法](#リリース方法)
  - [フォルダ構成/ファイル構成](#フォルダ構成/ファイル構成)
  - [自動テスト構成](#自動テスト構成)

# 結論

## 参考になった箇所
- コンポーネントがjsxをreturnしている箇所が小分けにされていて見やすかった。viewの責務が散らばる云々とか無関係に見やすいと思った。クラスコンポーネントだからよりそう感じるのかもしれない。これをいちいち別コンポーネントにするのもまた面倒だし…って思ったらこのようにしたほうがいいのかも。
  ```tsx
  //Avatar.jsx
  ...
  get avatarChildren() {
    const { src, alt, loaded, children } = this.props
    if (src && loaded) {
      return <img alt={alt} src={src} width={this.size} height={this.size} />
    } else if (children) {
      return children
    } else if (alt) {
      return this.initials
    }
  }

  ...
  
  return (
    <div
      id={this.state.id}
      style={{ ...style, ...sizeStyles }}
      className={`${this.getClasses('avatar')} ${className}`}
    >
      {this.avatarChildren}
    </div>
  )
  ```
  ```tsx
  //CardHeader.jsx
  ...

  render() {
    const {
      style,
      title,
      avatar,
      action,
      children,
      subtitle,
      className
    } = this.props
    const cardTitle = passDownProp(title, this.props, CARD_HEAD_PASS_DOWN)
    const cardAvatar = passDownProp(avatar, this.props, CARD_HEAD_PASS_DOWN)
    const cardAction = passDownProp(action, this.props, CARD_HEAD_PASS_DOWN)
    const cardSubTitle = passDownProp(subtitle, this.props, CARD_HEAD_PASS_DOWN)
    const cardChildren = passDownProp(
      children,
      this.props,
      CARD_CHILD_PASS_DOWN
    )

    return (
      <div style={style} className={`${this.getClass('wrapper')} ${className}`}>
        {cardAvatar || cardTitle || cardSubTitle || cardAction ? (
          <div className={this.getClass('content')}>
            <div className={this.getClass('content-left')}>
              {cardAvatar ? (
                <div className={this.getClass('avatar')}>{cardAvatar}</div>
              ) : null}
              <div>
                {cardTitle}
                {cardSubTitle}
              </div>
            </div>
            {cardAction}
          </div>
        ) : null}
        {cardChildren}
      </div>
    )
  }
  ```

  ```tsx
  // Carousel.tsx
  ...
  render() {
    const items = this.carouselItems
    const { style, className, hideDelimiters } = this.props
    return (
      <div
        style={{ ...this.styles, ...style }}
        className={`${this.getClasses('main')} ${className}`}
      >
        <div className={this.getClasses('nu-carousel-container')}>{items}</div>
        {hideDelimiters ? null : (
          <div className={this.getClasses('nu-carousel-controls')}>
            {this.getDelimiters(items)}
          </div>
        )}
        {this.nextIcon}
        {this.prevIcon}
      </div>
    )
  }
  ```
- componentsフォルダ配下はfeature（？）毎に分けずに並列。ただ、hocs、transitionsフォルダは分かれている。並列のほうが公式ドキュメントのリンクと並びが同じで分かりやすい気はする。小規模だからだとは思う。
- childrenに一括でonchangeをセットしている箇所がある
  ```tsx
  //Form.jsx
  ...
  render() {
    const { children } = this.props
    const formChildren = Children.map(children, (child) => {
      return cloneElement(child, {
        ...child.props,
        onChange: (e) => this.handleChange(e)
      })
    })
    return (
      <form onSubmit={this.handleSubmit} onChange={this.handleFormChange}>
        {formChildren}
      </form>
    )
  }
  ```

## 微妙だと思った箇所
- 自動テストが1ファイルでtruthyかしか見てない
- 生jsでコンポーネントが作られており、index.d.tsはおそらく手動管理

# 色々

## なぜ読もうとしたか
- 見た目がよかったのに、機能性も十分に思って興味が湧いた
- 簡単そうなので読めそうだった

## URL
https://github.com/AKAspanion/ui-neumorphism

## 使用ライブラリ
- react-transition-group

## 開発方法
特に工夫はない

## リリース方法
不明。package.jsonで管理はされてない

## フォルダ構成/ファイル構成

/
```js
example // 公式ドキュメントのソースコード
src // ソースコード
```

src/
```js
assets // ファイルでなく定数置き場
components // コンポーネント集
hocs // HOC集。ユーティリティ的なもの
transitions // アニメーション用コンポーネント集。react-transition-groupを使っている
util // ユーティリティ
```

components/
```
コンポーネントごとにフォルダを並列に作っている。ただ、さすがにinputとかtypography（このライブラリではH1~H6とかも含めてる概念になってる）とかは内部でいくつかのコンポーネントを作っている
```

## 自動テスト構成
index.test.jsという一つのファイルで、単にtruthyであることを各コンポーネントで確認しているだけ
```js
//例
describe('Typography', () => {
  it('is truthy', () => {
    expect(Typography).toBeTruthy()
  })
})
```
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

## 参考になった箇所

- 元となったruby版の方に、プラグインの説明がある。 <https://github.com/markdownlint/markdownlint/blob/main/docs/creating_rules.md>
- common mark specというのがある <https://spec.commonmark.org/>
- markdown-it（mdパーサー。toHtml()メソッドあり）に依存している。
  - markdown-itを拡張して自分のブログやサービス用のエディタのライブラリとして使ってるパターンがたまにあるっぽい。zenn-markdown-htmlとかもそう。
- vscode版は別リポジトリだった。でも内部ではこのmarkdownlintが使われているっぽい。　<https://github.com/DavidAnson/vscode-markdownlint>
  - こちらは <https://github.com/DavidAnson/vscode-markdownlint/blob/main/extension.js> の `function lint`が実際のリント処理をしているっぽい。そこで各行にwarnテキストなどを出しているっぽい？

    ```js
    task = markdownlintWrapper(document) //リント処理
      .then((results) => {
        const {activeTextEditor} = vscode.window;
        for (const result of results) { 
          // Create Diagnostics
          const lineNumber = result.lineNumber;
          const focusMode = applicationConfiguration[sectionFocusMode];
          const focusModeRange = (!Number.isInteger(focusMode) || (focusMode < 0)) ?
            0 :

          ...
            let range = document.lineAt(lineNumber - 1).range;  //対象の行番号
            if (result.errorRange) {
              const start = result.errorRange[0] - 1;
              const end = start + result.errorRange[1];
              range = range.with(range.start.with(undefined, start), range.end.with(undefined, end));
            }
            const diagnostic = new vscode.Diagnostic(range, message, vscode.DiagnosticSeverity.Warning); //分析メッセージを作成
            diagnostic.code = ruleInformationUri ?
              {
                "value": ruleName,
                "target": ruleInformationUri
              } :
              ruleName;
            diagnostic.source = extensionDisplayName;
            // @ts-ignore
            diagnostic.fixInfo = result.fixInfo; //fixするための情報を追加
            diagnostics.push(diagnostic);  //結果をエディタに出す？
    ```

  - ちなみにvscodeのエディタイベントはこんな感じのが例えばあるっぽい

    ```js
    // https://github.com/DavidAnson/vscode-markdownlint/blob/main/extension.js

    ... 

    // Handles the onDidOpenTextDocument event
    function didOpenTextDocument (document) { //テキストを開いたら
      if (isMarkdownDocument(document)) { //マークダウンなら
        lint(document);  //リント発火
        suppressLint(document);
      }
    }

    // Handles the onDidChangeTextDocument event
    function didChangeTextDocument (change) { //テキストに変更があれば
      const document = change.document;
      if (isMarkdownDocument(document) && (getRun(document) === "onType")) {
        requestLint(document);
      }
    }    
    ```

  - ちなみにそれらの関数は、activateという関数で実際にvscodeに登録されている感じっぽい

    ```js
    function activate (context) {
      // Create OutputChannel
      outputChannel = vscode.window.createOutputChannel(extensionDisplayName);
      context.subscriptions.push(outputChannel);  
      // Get application-level configuration
      getApplicationConfiguration();  
      // Hook up to workspace events
      context.subscriptions.push(
        // 以下らへん！
        vscode.window.onDidChangeActiveTextEditor(didChangeActiveTextEditor),
        vscode.window.onDidChangeTextEditorSelection(didChangeTextEditorSelection),
        vscode.window.onDidChangeVisibleTextEditors(didChangeVisibleTextEditors),
        vscode.workspace.onDidOpenTextDocument(didOpenTextDocument),
        vscode.workspace.onDidChangeTextDocument(didChangeTextDocument),
        vscode.workspace.onDidSaveTextDocument(didSaveTextDocument),
        vscode.workspace.onDidCloseTextDocument(didCloseTextDocument),
        vscode.workspace.onDidChangeConfiguration(didChangeConfiguration),
        vscode.workspace.onDidGrantWorkspaceTrust(didGrantWorkspaceTrust)
      );

    ...

    // これ！
    module.exports.activate = activate;

    ```

- snapshotsフォルダがあるということはスナップショットテストがあるっぽい。こういう系のライブラリなら特にスナップショットテストは有用だと思った。
- `markdownlint-rule-helpers`というnpmライブラリを内部で作っている。これはnpmにも公開していない。モノレポっぽくしている。いわゆる泥団子にならなくて良い気がする。
- eslint-plugin-regexp みたいなのがあるのか。今作ってるやつに使えるかも
- プラグイン形式でリントを実行している。プラグインは久美子みのは「md番号.js」的な形式で別ファイル化されている。

  ```js
  // @ts-check

  "use strict";

  const { addErrorDetailIf, filterTokens } = require("../helpers");

  module.exports = {
    "names": [ "MD001", "heading-increment", "header-increment" ],
    "description": "Heading levels should only increment by one level at a time",
    "tags": [ "headings", "headers" ],
    "function": function MD001(params, onError) {
      let prevLevel = 0;
      filterTokens(params, "heading_open", function forToken(token) {
        const level = Number.parseInt(token.tag.slice(1), 10);
        if (prevLevel && (level > prevLevel)) {
          addErrorDetailIf(onError, token.lineNumber,
            "h" + (prevLevel + 1), "h" + level);
        }
        prevLevel = level;
      });
    }
  };
  ```  

  - ↑の`function`が`markdownlint.js`で呼ばれている。

    ```js
        const invokeRuleFunction = () => rule.function(params, onError);
        ...
        invokeRuleFunction();
    ```

- markdown-itは `const tokens = md.parse(content, {});` のようにtokenにパースする目的で使われているっぽい。でそのパースされたtokenにリントチェックを↑のようにforEach的にかけて回っているっぽい。
  - markdown-it用のプラグインがたくさんある <https://www.npmjs.com/search?q=keywords:markdown-it-plugin>。これらは.md拡張構文リスト的なものと見て良い気がする。

## 微妙だと思った箇所

- 基本tsでなくjsで書かれているっぽい？
- モノレポツールを使わずに内部パッケージを頑張っているっぽい？

## 他気になった箇所

- clone-test-repos系のスクリプトは何をしたいのか？なぜcloneだけしているのかｆ
- なんでciをcronでも実行させているのか。prとpushがあればいらないのでは、？そこがほしい用の知識が無い

  ```yml
  on:
    pull_request:
    push:
    schedule:
      - cron: '30 12 * * *'
    workflow_dispatch:
  ```

- 以下の行はなんなのか。「最小限のマシンであってもワーカーは8つは耐えられるし、8つはほしいよ」的なことかな、？こんなことをやるのか。

```js
  if (synchronous) {
    while (!done) {
      lintWorker();
    }
  } else {
    // Testing on a Raspberry Pi 4 Model B with an artificial 5ms file access
    // delay suggests that a concurrency factor of 8 can eliminate the impact
    // of that delay (i.e., total time is the same as with no delay).
    lintWorker();
    lintWorker();
    lintWorker();
    lintWorker();
    lintWorker();
    lintWorker();
    lintWorker();
    lintWorker();
  }
```

## 使い方

node.jsまたはブラウザで使えるやつ。importしたものを叩くだけっぽい。ちゃんと行単位でエラーやwarnを表示してくれる。
<https://github.com/DavidAnson/markdownlint#usage>

CLI、エディタ、github actions用はそれ用の統合版があるっぽい。
<https://github.com/DavidAnson/markdownlint#related>

## なぜ読もうとしたか

## URL

<https://github.com/DavidAnson/markdownlint>

## 使用ライブラリ

- markdown-it
  - いくつかそれ用のプラグインも使っていた。

    ```json
    "markdown-it-footnote": "3.0.3",
    "markdown-it-for-inline": "0.1.1",
    "markdown-it-sub": "1.0.0",
    "markdown-it-sup": "1.0.0",
    "markdown-it-texmath": "1.0.0",
    ```

- 内部でnpmパッケージ（npmに公開はしていない）を作って使っていた
  - `markdownlint-rule-helpers`
- 他普通のやつ。
  - テストランナーはava
  - カバレッジはc8
  - eslint
  - tv4でjsonスキーマバリデートしている？

## 開発方法

- 他ライブラリと同じ用にCONTRIBUTING.mdにコントリビュートの流れが書かれている。

## リリース方法

- 普通のやり方だと思う

## フォルダ構成やファイル構成

- helpersというフォルダが内部でライブラリ扱いでpackage.josnからインストールしようとしているのが参考になった。
- メインのソースコード置き場はsrcでなくlibだった。そこを見に行くのはpackage．jsonに記載されている。
- dictionary.txtがあった。これはtypo防止のCIで使われている。個人開発のウェブアプリではここまでしなくてもいいとは思うけど、こういうライブラリではあってもいいのかも。いろんなcontributor現れるかもだしみんながtypoチェッカー的なプラグイン使って開発するとも限らないので。

## 自動テスト構成

いつものやつ。
package.jsonのscriptsの中のciという項目がメインっぽい。

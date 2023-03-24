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

- 以下のQA手法が良さそう。真似した実装はできそう
  - dependency-cruiserとgit diffを組み合わせた、diffのあるファイルに依存しているページ一覧をPRにコメントする機能
  - graphql-inspectorを使い、スキーマの変更をBreaking, dangerous, safeの３つに分けてPRにコメントする機能
  - 差分が多くあったページごとにgzip圧縮した際のバンドルサイズをPRにコメントする機能
  - JestのcoverageThresholdを使い、最低カバレッジを満たさないPRに警告を出す機能
- 自動テストのカバレッジをPRの必須項目にするのは考え方として良さそう。だけどコストがかかりそうなのでお金のある会社でないと現実的ではない方法論ではありそう。

## 微妙だと思った箇所

## 他気になった箇所

## 使い方

## なぜ読もうとしたか

- 自動テストに興味があるから

## URL

<https://engineering.mercari.com/blog/entry/20220218-mercari-shops-automation/>

## 使用ライブラリ

## 開発方法

## リリース方法

## フォルダ構成やファイル構成

## 自動テスト構成

## その他

「dependency-cruiserとgit diffを組み合わせた、diffのあるファイルに依存しているページ一覧をPRにコメントする機能」のようなものは再現しようとしてみた。のようなものです。

結果、まだ納得行くのができていない。まず手元で試す限りcruise(~)関数の結果が正しくない気がする（dependenciesが少ないような実行結果に鳴ってしまう。それがどうしても修正できなかった。そんなものなのか、？）。

dependenciesでなくdependentsを使うようにしてみたソースは以下。ただし以下は

- ファイルが複数のexportを持っていたら破綻する気がする（例えばファイル内の関数Aに差分ができただけなのに、同じファイル内にある関数B〜Zをimportしているファイルまでもがdependentsに現れてしまう（のかも。まだ詳しく見てない）。仮にそうなら、1ファイル1export的なルールを定めるとかしないと正しく依存関係を追えない。あと、同じ理由でバレル的なファイルがあるとたぶん破綻する（のかも。まだ詳しく見てない））

という欠陥があるかもなため解消が必要。

以下に途中のソースを上げる。

```ts
// 準備: yarn add -D dependency-cruiser typescript
// 実行: ts-node --project tsconfig.script.json このファイル名.ts
//       ※スクリプトを実行できるtsconfigのファイルを作ってそれを指定している

import { cruise, IReporterOutput } from "dependency-cruiser";
import * as dependencyCruiserConfig from "./.dependency-cruiser.js";

// TODO: これだとファイル単位でしか見てないので、変更かけた関数単位などでも依存関係を見れるようにしたいな？
// TODO: CLIで使えるようにして、かつ以下は引数で渡せるようにする

const TARGET_FILE = "src/好きなパス.ts";
const ARRAY_OF_FILES_AND_DIRS_TO_CRUISE: string[] = [
  "src/**/*.ts",
  "src/**/*.tsx",
];
const INCONSISTENT_PREFIX_SPELLINGS = ["src", "@"];

// 雑音を無視してマッチするか判定
const isMatchedPath = (
  targetFilePath: string,
  comparedFilePath: string,
  {
    inconsistentPrefixSpellings = [],
  }: {
    inconsistentPrefixSpellings: string[];
  },
) => {
  const omitNoiseForCompare = (filePath: string) => {
    return filePath
      .replace(new RegExp(`^(${inconsistentPrefixSpellings.join("|")})`), "") // omit noisy prefix
      .replace(/\.[^/.]+$/, ""); // omit ext
  };

  return (
    omitNoiseForCompare(targetFilePath) ===
    omitNoiseForCompare(comparedFilePath)
  );
};

const setDependantsRecursively = (mutableTree: object, targetFile: string) => {
  const cruiseResult: IReporterOutput = cruise(
    ARRAY_OF_FILES_AND_DIRS_TO_CRUISE,
    dependencyCruiserConfig.options,
  );
  if (typeof cruiseResult.output === "string") {
    console.log(444, cruiseResult.output);
  } else {
    cruiseResult.output.modules.forEach((module) => {
      if (
        isMatchedPath(module.source, targetFile, {
          inconsistentPrefixSpellings: INCONSISTENT_PREFIX_SPELLINGS,
        })
      ) {
        module.dependents.forEach((dependent) => {
          mutableTree[targetFile] ??= {};
          mutableTree[targetFile][dependent] ??= {};
          setDependantsRecursively(mutableTree[targetFile], dependent);
        });
      }
    });
  }
};

const dependentTree = {};
setDependantsRecursively(dependentTree, TARGET_FILE);
console.log(JSON.stringify(dependentTree, null, 2));
```

でその欠陥を解消するには以下をやる必要がありそう。

- 差分をファイル単位だけで見るのでなく、スコープ単位でも見る（例えばファイルAに関数A〜Zがあったとして、関数Bだけいじった時は「ファイルAに依存するファイル」をdependency cruiseするのでなく、「ファイルAから関数Bをimportしているファイル」をdependency cruiseするようにしないといけない。これはdependency-cruiserだとできないかもなので、typescript compiler apiを使うとかしか無いのかも）

追記：これだけならdependency-treeというライブラリを使えば同じことできるっぽい
```ts
var dependencyTree = require("dependency-tree");

// Returns a dependency tree object for the given file
var tree = dependencyTree({
  filename: "./src/好きなファイルパス.tsx",
  directory: "./src",
  // requireConfig: 'path/to/requirejs/config', // optional
  // webpackConfig: 'path/to/webpack/config', // optional
  tsConfig: "./tsconfig.json",
  // nodeModulesConfig: {
  //   entry: 'module'
  // }, // optional
  filter: (path) => path.indexOf("node_modules") === -1, // optional
  // nonExistent: [], // optional
  // noTypeDefinitions: false // optional
});

console.log(tree);

// Returns a post-order traversal (list form) of the tree with duplicate sub-trees pruned.
// This is useful for bundling source files, because the list gives the concatenation order.
// Note: you can pass the same arguments as you would to dependencyTree()
// var list = dependencyTree.toList({
//   filename: 'path/to/a/file',
//   directory: 'path/to/all/files'
// });

```

というのは一旦別として、以下はjsファイル内で呼ばれている関数名一覧をconsole.logするスクリプト

```ts
import acorn from "acorn";
import { simple } from "acorn-walk";

function getCalledFunctionNames(code: string) {
  const calledFunctionNames = [] as any[];

  // プログラムをパースしてASTを取得
  const ast = acorn.parse(code, { ecmaVersion: 2021 });

  // ASTを走査して、CallExpressionノードを探す
  simple(ast, {
    CallExpression: (node: any) => {
      if (node.callee.type === "Identifier") {
        calledFunctionNames.push(node.callee.name);
      }
    },
  });

  // 重複した関数名を削除して、結果を配列に格納
  return Array.from(new Set(calledFunctionNames));
}

const functionCode = `
g()

function a() {
  const e = 1 + 1;
  b();
  function c() {
    d();
  }
  return e;
}

a()
`;

//TODO: TSに対応したいところ…
const calledFunctionNames = getCalledFunctionNames(functionCode);
console.log(calledFunctionNames); //[ 'g', 'b', 'd', 'a' ]
```

上は、jsのみしか対応してない。
以下はts対応版。

```ts
import ts, { SourceFile } from "typescript";

//TODO: コマンドから渡せるようにする
const FILE_PATH = "./astts-source.ts";         //これを準備してください（パースする対象のtsファイル）。適当なやつでOK。

const getSourceFile = (filePath: string): SourceFile | undefined => {
  const program = ts.createProgram([filePath], {});
  program.getTypeChecker(); // HACK: なぜかこの行が無いとエラーになる
  const source = program.getSourceFile(filePath);
  return source;
};

// 厳密には関数のみでなくコンストラクタも含む
const getCalledFunctionNames = (source: SourceFile): string[] => {
  if (!source) {
    throw new Error("Source file not found");
  }

  const calledFunctionNames: string[] = [];
  function traverse(node: ts.Node) {
    if (node.kind === ts.SyntaxKind.CallExpression) {
      const functionName = (node.getChildAt(0) as ts.Identifier)?.getText();
      calledFunctionNames.push(functionName);
    }
    ts.forEachChild(node, traverse);
  }

  ts.forEachChild(source, (node) => {
    traverse(node);
  });

  return calledFunctionNames;
};

const getFunctionCalledScope = (source: SourceFile, functionName: string) => {
  if (!source) {
    throw new Error("Source file not found");
  }

  const calledFunctionNames: string[] = [];
  function traverse(node: ts.Node) {
    if (node.kind === ts.SyntaxKind.CallExpression) {
      // const functionName = (node.getChildAt(0) as ts.Identifier)?.getText();
      // calledFunctionNames.push(functionName);
    }
    ts.forEachChild(node, traverse);
  }

  ts.forEachChild(source, (node) => {
    traverse(node);
  });

  return calledFunctionNames;
};

const sourceFile = getSourceFile(FILE_PATH);
if (sourceFile) {
  console.log(getCalledFunctionNames(sourceFile)); //[ 'b', 'd', '(() => {\n    b();\n  })', 'b', 'c', 'b' ]
} else {
  console.log("Source file not found");
}

```

↑が参照している`astts-source.ts`はこんな感じ

```ts
const b = () => {};
const d = () => {};

function a() {
  const e = 1 + 1;

  b();

  function c() {
    d();
  }

  (() => {
    b();
  })();

  c();

  return e;
}

b();
```

↑の↑のソース内のgetFunctionCalledScopeは未実装。本来の目的の一つである関数が呼ばれてるスコープを取得したいのをやりたいやつ。

で以下はそれをできるようにしたっぽいやつ。

```ts
import ts, { SourceFile } from "typescript";

//TODO: コマンドから渡せるようにする
const FILE_PATH = "./astts-source.ts";

const getSourceFile = (filePath: string): SourceFile | undefined => {
  const program = ts.createProgram([filePath], {});
  program.getTypeChecker(); // HACK: なぜかこの行が無いとエラーになる
  const source = program.getSourceFile(filePath);
  return source;
};

// 厳密には関数のみでなくコンストラクタも含む
const getCalledFunctionNames = (source: SourceFile): string[] => {
  if (!source) {
    throw new Error("Source file not found");
  }

  const calledFunctionNames: string[] = [];
  function traverse(node: ts.Node) {
    if (node.kind === ts.SyntaxKind.CallExpression) {
      const functionName = (node.getChildAt(0) as ts.Identifier)?.getText();
      calledFunctionNames.push(functionName);
    }
    ts.forEachChild(node, traverse);
  }

  ts.forEachChild(source, (node) => {
    traverse(node);
  });

  return calledFunctionNames;
};

function getParentFunction(
  node: ts.Node,
): ts.FunctionDeclaration | ts.FunctionExpression | undefined {
  let parent = node.parent;
  while (parent) {
    if (ts.isFunctionDeclaration(parent) || ts.isFunctionExpression(parent)) {
      return parent;
    }
    parent = parent.parent;
  }
  return undefined;
}
const getFunctionCalledScopes = (
  source: SourceFile,
  functionName: string,
): (string | "<global>")[] => {
  if (!source) {
    throw new Error("Source file not found");
  }

  const parentFunctionNames: string[] = [];
  function traverse(node: ts.Node) {
    if (node.kind === ts.SyntaxKind.CallExpression) {
      if (node.getChildAt(0).getText() === functionName) {
        const parentFunctionName = getParentFunction(node)?.name?.text;
        parentFunctionNames.push(parentFunctionName ?? "<global>");
      }
    }
    ts.forEachChild(node, traverse);
  }

  ts.forEachChild(source, (node) => {
    traverse(node);
  });

  return parentFunctionNames;
};

const sourceFile = getSourceFile(FILE_PATH);
if (sourceFile) {
  console.log(
    "指定ソース内で実行されている関数名一覧:",
    getCalledFunctionNames(sourceFile),
  );
  console.log(
    "指定ソース内で指定関数を実行している関数名一覧:",
    getFunctionCalledScopes(sourceFile, "b"),
  );
  // TODO: できればexportしてるものに関するものだけ出力したい
} else {
  console.log("Source file not found");
}
```

結果は

```sh
指定ソース内で実行されている関数名一覧: [ 'b', 'd', '(() => {\n    b();\n  })', 'b', 'c', 'b' ]
指定ソース内で指定関数を実行している関数名一覧: [ 'a', 'a', '<global>' ]
```

一応それなりに取得できてる気はする。ただ、これだと

- exportしている関数だけ取得したい

って思ってしまった（追記：今思えば別にその必要は無さそう。なぜならば、「dependency-cruiserで見つけた指定ファイルのdependentsのファイル内で、指定の関数が実行されている場所」を探すというロジックだけで「関数が実行されてない/されている」の結果は手に入るので（うまく説明できない）。それがexportされているものなのかどうかは見る必要がそもそも無いかもという）

（追記：じゃあ↑と組み合わせてしまえば依存関係がひとまず関数においては取得できるのかも。大体は）

以下は一応exportしているやつのみ取得するスクリプトの途中のやつ。やってみたので残しておく。

```ts
import * as ts from "typescript";
import * as path from "path";

// exportしているもの一覧を取得
// TODO: typeとかが取得できてない。export * from とかも無理かも。
function listExports(file: string): string[] | undefined {
  const program = ts.createProgram([file], {});
  const checker = program.getTypeChecker();
  const sourceFile = program.getSourceFile(file);

  if (!sourceFile) {
    console.error(`Could not find source file: ${file}`);
    return;
  }

  const exportedSymbols: string[] = [];
  ts.forEachChild(sourceFile, (node) => {
    if (
      ts.isExportDeclaration(node) ||
      ts.isExportAssignment(node) ||
      ts.isExportSpecifier(node)
    ) {
      if (ts.isExportDeclaration(node)) {
        const exportedNames = (node.exportClause as any).elements.map(
          (element) => element.name.getText(),
        );
        exportedSymbols.push(...exportedNames);
      } else {
        const symbol = checker.getSymbolAtLocation((node as any).expression);
        if (symbol) {
          exportedSymbols.push(symbol.getName());
        }
      }
    } else if (ts.isNamedExports(node)) {
      console.log((node as any).exportClause);
      node.elements.forEach((element) => {
        const symbol = checker.getSymbolAtLocation(element.name);
        if (symbol) {
          exportedSymbols.push(symbol.getName());
        }
      });
    } else if (ts.isVariableStatement(node)) {
      // get export declaration
      const exportDeclaration = node.modifiers?.find((modifier) => {
        return modifier.kind === ts.SyntaxKind.ExportKeyword;
      });
      if (exportDeclaration) {
        // get symbol
        const symbol = checker.getSymbolAtLocation(
          node.declarationList.declarations[0].name,
        );
        if (symbol) {
          exportedSymbols.push(symbol.getName());
        }
      }
    }
  });

  return exportedSymbols;
}

const filePath =
  "./src/好きなファイルへのパス.tsx";
const absolutePath = path.resolve(filePath);
console.log(listExports(absolutePath));
```


という形でいろいろやってみてるけど面倒そうな割に「いや影響範囲の分かりやすいソースコードとspecがあれば別にこういうスクリプト無くてもほぼ困ること無いな」と思って一旦ストップになった。あとは他人から求められたら進めるかも。

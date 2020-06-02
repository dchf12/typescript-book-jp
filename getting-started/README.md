# TypeScript入門

* [TypeScript入門](./#typescriptwomeyou)
* [TypeScriptのバージョン](./#typescriptnobjon)

## TypeScript入門

TypeScriptは、 最終的にJavaScriptにコンパイルされます。実際に実行されるのは、JavaScriptです(ブラウザでも、サーバーでも同じです)。よって、TypeScriptを利用するには、次のものが必要です：

* TypeScriptコンパイラ (OSSの[ソース](https://github.com/Microsoft/TypeScript/)および、[NPM](https://www.npmjs.com/package/typescript)で利用可能です)
* TypeScriptのエディタ (メモ帳を使うことも可能ですが、 [vscode](https://code.visualstudio.com/) と [私が作成したプラグイン](https://marketplace.visualstudio.com/items?itemName=basarat.god) を併用することをお勧めします。また、[様々なIDE](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support) がサポートされています。)

### TypeScriptのバージョン

安定版のTypeScriptコンパイラを使用する代わりに、本書ではバージョン番号に関連付けられていない多くの新規要素を紹介します。
コンパイラのテスト環境は時間が経過するほど多くのバグを見つけるため、私は一般的に夜間ビルド(nightly version)の最新版を使用することを推奨します。

次のコマンドでインストールできます。npmコマンドは[Node.js](https://nodejs.org/ja/)をインストールすると利用できます。

```text
npm install -g typescript@next
```

これで、`tsc`コマンドは最新かつ最高のものになります。IDEでもそれを利用することが可能です。例:

* vscodeで利用するTypeScriptのバージョンのパスを `.vscode/settings.json`で指定できます

```javascript
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}
```

### ソースコード取得

この書籍のソースコードは、githubのリポジトリ [https://github.com/basarat/typescript-book/tree/master/code](https://github.com/basarat/typescript-book/tree/master/code) にあります。 ほとんどのコードサンプルは、vscodeにコピーしてそのまま実行できます。追加設定が必要なコードサンプル\(例：npmモジュール\)では、そのコードを表示する前にコードサンプルのパスにリンクします。例:

`this/will/be/the/link/to/the/code.ts`

```typescript
// 議論中のコード
```

開発設定をして、TypeScriptの構文を見ていきましょう。

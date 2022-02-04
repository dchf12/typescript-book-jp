* [矢印機能](#arrow-functions)
* [ヒント：矢印機能が必要](#tip-arrow-function-need)
* [Tip：矢印機能の危険性](#tip-arrow-function-danger)
* [ヒント： `this`を使用するライブラリ](#tip-arrow-functions-with-libraries-that-use-this)
* [ヒント：矢印関数の継承](#tip-arrow-functions-and -herher)
* [ヒント：クイックオブジェクトリターン](#tip-quick-object-return)

### 矢印関数

* 太い矢印*(薄い矢印で `=>`は太い矢印である)とlambda関数*(他の言語のため)とも呼ばれています。別の一般的に使用される機能は太い矢印関数 `()=> something`です。 *太った矢印*の動機は：
1. あなたは `function`をタイプし続ける必要はありません
2. それは「これ」の意味を語彙的に捕捉する
2. 「議論」の意味を語彙的に捕捉する

機能的であると主張する言語の場合、JavaScriptでは `function`をかなり多く入力する傾向があります。太い矢印を使用すると、関数を簡単に作成できます
```ts
var inc = (x)=>x+1;
```
「これは、従来、JavaScriptの苦労してきたことです。賢明な人は、「私はJavaScriptが嫌いですが、これは「このすべて」の意味をあまりにも簡単に失う傾向があるからです。脂肪族の矢は、周囲の文脈から `this`の意味を捉えて修正します。この純粋なJavaScriptクラスを考えてみましょう：

```ts
function Person(age) {
    this.age = age;
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 1, should have been 2
```
このコードをブラウザで実行すると、関数内の `this`は`window`を指すようになります。なぜなら、 `window`は`growOld`関数を実行するものになるからです。修正は、矢印関数を使用することです：
```ts
function Person(age) {
    this.age = age;
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
これがなぜ機能するのかは、関数本体の外側からの矢印関数によって 'this'への参照が捕捉されるためです。これは次のJavaScriptコードと同じです(TypeScriptを使用していない場合は、自分で書くと思います)。
```ts
function Person(age) {
    this.age = age;
    var _this = this;  // capture this
    this.growOld = function() {
        _this.age++;   // use the captured this
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
TypeScriptを使用しているので、構文が甘くなり、矢印をクラスと組み合わせることができます。
```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

> [このパターンについての甘いビデオ🌹](https://egghead.io/lessons/typescript-make-usages-of-this-safe-in-class-methods)

#### ヒント：矢印機能が必要
簡潔な構文のほかに、他の人に関数を渡す場合は太い矢印を使用する必要があります。効果的に：
```ts
var growOld = person.growOld;
// Then later someone else calls it:
growOld();
```
あなたがそれを自分で呼び出す場合、つまり
```ts
person.growOld();
```
`this`は正しい呼び出しコンテキスト(この例では`person`)になります。

#### ヒント：矢印機能の危険

実際に `this`*を呼び出しコンテキストにしたい場合は*矢印関数*を使用しないでください。 jquery、underscore、mochaなどのライブラリで使用されるコールバックのケースです。ドキュメンテーションが `this`の関数を記述している場合は、たぶん太い矢印の代わりに`function`を使うべきでしょう。同様に、 `arguments`を使用する予定の場合は、矢印関数を使用しないでください。

#### ヒント： `this`を使用するライブラリを持つArrow関数
多くの図書館がこれを行っている。 `jQuery`iterables(ある例ではhttps://api.jquery.com/jquery.each/)は` it`を使って現在反復中のオブジェクトを渡します。この場合、渡されたライブラリーに `this`だけでなく周囲のコンテキストにもアクセスしたい場合は、矢印関数がない場合と同様に`_self`のような一時変数を使用してください。

```ts
let _self = this;
something.each(function() {
    console.log(_self); // the lexically scoped value
    console.log(this); // the library passed value
});
```

#### ヒント：矢印機能と継承
矢印の関数はクラスのプロパティとして継承で正常に動作します：

```ts
class Adder {
    constructor(public a: number) {}
    add = (b: number): number => {
        return this.a + b;
    }
}
class Child extends Adder {
    callAdd(b: number) {
        return this.add(b);
    }
}
// Demo to show it works
const child = new Child(123);
console.log(child.callAdd(123)); // 246
```

しかし、子クラスの関数をオーバーライドしようとすると、 `super`キーワードでは動作しません。プロパティは `this`に行きます。このような関数は `super`(`super`はプロトタイプメンバーのみで動作します)への呼び出しには参加できません。メソッドを子にオーバーライドする前に、メソッドのコピーを作成することで簡単に回避できます。

```ts
class Adder {
    constructor(public a: number) {}
    // This function is now safe to pass around
    add = (b: number): number => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Create a copy of parent before creating our own
    private superAdd = this.add;
    // Now create our override
    add = (b: number): number => {
        return this.superAdd(b);
    }
}
```

### ヒント：クイックオブジェクトリターン

単純なオブジェクトリテラルを返す関数が必要な場合もあります。しかし、何かのような

```ts
// WRONG WAY TO DO IT
var foo = () => {
    bar: 123
};
```
JavaScriptランタイム(JavaScript仕様の原因)によって* JavaScript Label *を含む*ブロック*として解析されます。

> これが意味をなさない場合は、とにかく "未使用のラベル"と言っているタイプスクリプトからの素晴らしいコンパイラエラーが発生するので、心配しないでください。ラベルは古い(そしてほとんどは使用されていない)JavaScript機能で、現代のGOTO(経験豊富な開発者が悪いと考える🌹)として無視することができます。

`()`でオブジェクトのリテラルを囲むことで修正できます：

```ts
// Correct 🌹
var foo = () => ({
    bar: 123
});
```

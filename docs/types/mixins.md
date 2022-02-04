# 混合クラス(Mixins)

TypeScript(およびJavaScript)クラスは、厳密な単一継承をサポートします。あなたはこれをすることはできません：

```ts
class User extends Tagged, Timestamped { // ERROR : no multiple inheritance
}
```

再使用可能なコンポーネントからクラスを構築する別の方法は、mixinと呼ばれるより単純な部分クラスを組み合わせてそれらを構築することです。

そのアイデアは、機能を得るためにクラスAを拡張する*クラスAの代わりに単純です。*関数BはクラスA を取り、この追加された機能を持つ新しいクラスを返します。関数`B`はミックスインです。

> [mixinは]
>
> 1. コンストラクタをとり、
> 1. コンストラクタを拡張し、新しい機能を持つクラスを作成する
> 1. 新しいクラスを返す

完全な例

```ts
// Needed for all mixins
type Constructor<T = {}> = new (...args: any[]) => T;

////////////////////
// Example mixins
////////////////////

// A mixin that adds a property
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

// a mixin that adds a property and methods
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActivated = false;

    activate() {
      this.isActivated = true;
    }

    deactivate() {
      this.isActivated = false;
    }
  };
}

////////////////////
// Usage to compose classes
////////////////////

// Simple class
class User {
  name = '';
}

// User that is Timestamped
const TimestampedUser = Timestamped(User);

// User that is Timestamped and Activatable
const TimestampedActivatableUser = Timestamped(Activatable(User));

////////////////////
// Using the composed classes
////////////////////

const timestampedUserExample = new TimestampedUser();
console.log(timestampedUserExample.timestamp);

const timestampedActivatableUserExample = new TimestampedActivatableUser();
console.log(timestampedActivatableUserExample.timestamp);
console.log(timestampedActivatableUserExample.isActivated);

```

この例を分解してみましょう。

## コンストラクタを取る

ミックスインはクラスを拡張し、新しい機能を追加します。したがって、*コンストラクタ*を定義する必要があります。次のように簡単です：

```ts
// Needed for all mixins
type Constructor<T = {}> = new (...args: any[]) => T;
```

## クラスを拡張して返します

とても簡単です：

```ts
// A mixin that adds a property
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```

これだけです🌹

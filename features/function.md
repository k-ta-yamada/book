# 関数 \(Functions\)

JavaScriptでは関数は第一級オブジェクトです。よって変数に代入したりすることも可能です。また筆記方法も複数存在し、TypeScriptもそれを継承しています。

## 関数の宣言

関数の宣言は主に3通りの方法があります。以下は全て同じ関数`increment()`を宣言しています。

### 名前付き関数 `(Normal functions)`

```typescript
function increment(num: number): number {
  return num + 1;
}
```

### 匿名関数 `(Anonymous functions)`

```typescript
const increment = function(num: number): number {
  return num + 1;
};
```

### 匿名かつアロー関数 `(Arrow functions)`

```typescript
const increment = (num: number): number => {
  return num + 1;
};
```

本書では関数は全て名前付き関数での記述となっています。

また、匿名かつアロー関数は、1行で戻り値を返却できる場合はさらに短縮することができます。

### 匿名かつアロー関数1行版

```typescript
const increment = (num: number): number => num + 1;
```

この時は`return`を**書いてはいけない**ので注意してください。ちなみに1行と書きましたが、厳密にはステートメントがひとつであれば改行しても問題ありません。

さらに、引数が1個である場合は`()`も省略できます。

### 匿名かつアロー関数1行かつ引数が1個版

```typescript
const increment = num => num + 1;
```

ただし、これができるのは引数が1個の時のみで、0個の時や複数ある時はできません。  
また、この時は引数と戻り値に対して**型をつけることができません**。

アロー関数の1行版でオブジェクトリテラルを返したい時はそのまま返すことができません。

```typescript
const func = () => {x: 1};

console.log(func());
// undefined
```

この時はオブジェクトリテラルを`()`で括ってください。

```typescript
const func = () => ({x: 1});

console.log(func());
// { x: 1 }
```

今では使う機会も減りましたが`generator`という特殊な関数を作ることもできます。ですがこの関数はアロー関数での表記を認めておらず、必ず`function*() {}`と書く必要があります。

## `this`

`function() {}`と`() => {}`で異なる点はいくつかありますが特に注意しなければならないのは`this`です。例えば以下のようなクラス`JetLag`を考えます。

```typescript
class JetLag {
  private message: string;
  
  public constructor(message: string) {
    this.message = message;
  }
  
  public replyFunction(ms: number): void {
    setTimeout(function() {
      console.log(this.message);
    }, ms);
  }

  public replyArrow(ms: number): void {
    setTimeout(() => {
      console.log(this.message);
    }, ms);
  }
}
```

このクラスのメソッド`replyFunction(), replyArrow()`は、どちらも指定したミリ秒後にコンストラクタで指定した文字列を表示するように見えますが`replyFunction()`に問題があります。

```typescript
const jetlag: JetLag = new JetLag('i can hear you later');

jetlag.replyFunction(10);
// -> 'this' implicitly has type 'any' because it does not have a type annotation.
// -> An outer value of 'this' is shadowed by this container.
jetlag.replyArrow(10);
// 'i can hear you later'
```

これは`function() {}`と`() => {}`で`this`が意味するコンテキストが違うために起こります。`() => {}`は宣言時に`this`であるものを使用するのに対して`function() {}`は実行時に`this`であるものを使用します。

### 不定な`this`をはっきりさせる

`function() {}`を使うときに`this`を確定させる方法として以下のふたつがあります。以下は上記クラスの`replyFunction()`を書き換えていると解釈してください。

#### 使いたい`this`を退避させる

一度`this`をほかの変数に代入してあとで呼び出します。

```typescript
public replyFunction(ms: number): void {
  const self: this = this;

  setTimeout(function() {
    console.log(self.message);
  }, ms);
}
```

#### `function() {}`の`this`を束縛する

`function`には`bind()`という関数があります。その関数に`function`の中で`this`として使用したい変数を引数に入れます。

```typescript
public replyFunction(ms: number): void {
  setTimeout(function() {
    console.log(this.message);
  }.bind(this), ms);
}
```

ただし、この方法だけではJavaScriptでは実行できるのですが、TypeScriptでは実行できません。この`bind()`を使って意図する動作を得るには後述する**引数の`this`**を併せて使う必要があります。

`() => {}`が実装される前まではほぼ必須だった`this`の取り扱いですが、現在ではそこまで必要ではなくなりました。

## 関数の型

匿名関数、匿名かつアロー関数では変数に代入していることからわかるように、関数も型による表現が可能です。

上記関数`increment()`では関数の型はこのようになります。

```typescript
(num: number) => number;
```

これは匿名かつアロー関数と少々宣言が異なります。厳密に言うと戻り値の位置が異なります。匿名かつアロー関数は、実体を除けば以下の形をしています。

```typescript
(num: number): number => {...};
```

オブジェクト風の書き方もあります。

```typescript
type Operate = {
  (num: number): number;
};
```

## 引数 `(Arguments)`

関数の入力値である引数は特殊なことをしない限り、要求する型の変数を、要求する数だけ入力しなければいけません。  
例えば原点との距離を求める以下の関数があったとします。

```typescript
function distance(p: Point): number {
  return (p.x ** 2 + p.y ** 2) ** (1 / 2);
}
```

なお、xy座標上の点を表す`Point`の定義は以下です。

```typescript
type Point = {
  x: number;
  y: number;
};
```

関数`distance()`は平面状にある点`(x, y)`の原点からの距離を返します。この関数を呼ぶ時は必ず引数の数、順番は揃わなければなりません。つまり以下のような関数呼び出しはできません。

### 引数が少ない

```typescript
distance();
// -> Expected 1 arguments, but got 0.
```

### 引数が多い

```typescript
distance(q1, q2);
// -> Expected 1 arguments, but got 2.
```

JavaScriptでは引数が少ない時はその引数には`undefined`が渡され、引数が多い場合は余分な引数は無視されるのですが、ここは大きな違いです。

## `Optional parameters, Default parameters`

引数を省略したいことがあります。その時は`Optional parameters`と`Default parameters`を使用することができます。

上記の関数`distance()`は、現在は与えられた座標を元に原点からの距離を計算していますが、これを2点の距離を計算できるようにしたいとします。すると上記の関数`distance()`は以下のようになります。

```typescript
function distance(p1: Point, p2: Point): number {
  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}

distance(q1, q2);
```

### `Optional parameters`

ここで、第2引数は省略可能にし、省略した場合は第1引数と原点の距離を返したいとします。これは`Optional parameters`を使用すると以下のように書けます。

```typescript
function distance(p1: Point, p2?: Point): number {
  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}

distance(q1, q2);
distance(q1);
```

引数の`p2`の右隣に`?`がつきました。これで`p2`は省略可能な引数となり5, 6行目のどちらの書き方も受け付けるようになります。

しかし、この`Optional parameters`は意味する型が少々変わります。内部的には`p2`は`Point`ではなく`Point | undefined`のユニオン型`(Union Types)`として解釈されます。ユニオン型の説明は先の章にあるため詳しい説明は譲りますが、ユニオン型は日本語で言うと**どれか**の意味です。

ユニオン型が与えられた時は、どちらの型にもあるプロパティ、メソッドでなければ使うことができません。当然ながら`undefined`には`x, y`というプロパティは存在しないため、上記のコードはTypeScriptに指摘されます。

この問題を解消したのが以下のふたつです。

#### 省略時の初期化処理を書く

```typescript
function distance(p1: Point, p2?: Point): number {
  let p0: Point | undefined = p2;
  
  if (p0 === undefined) {
    p0 = {
      x: 0,
      y: 0
    };
  }

  return ((p1.x - p0.x) ** 2 + (p1.y - p0.y) ** 2) ** (1 / 2);
}
```

省略時はどの値を使うかという処理が明文化されますが、後述の`Default parameters`がほぼ同じことをできます。これで実装できる場合は`Default paramteres`の使用を検討してください。

#### 処理を分ける

```typescript
function distance(p1: Point, p2?: Point): number {
  if (p2 === undefined) {
    return (p1.x ** 2 + p1.y ** 2) ** (1 / 2);
  }

  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}
```

`if`は`p2`が`undefined`の時に実行され、内部で`return`をしています。そのため`if`ブロックの下は`p2`が`Point`であることが確定します。そのためTypeScriptはユニオン型から普通の`Point`として解釈し、このように書くことができるようになります。

### `Point | undefined` との違い

`p2`の型が`Point | undefined`として解釈されるのなら、あえて`?`などという記号を新しく定義する必要などないのではと思われるかもしれませんが、明確な違いがあります。それは**呼び出し側で省略できるかどうかということ**です。上記の通り`Optional parameters`は省略が可能なのですが、`undefined`とのユニオン型であることを明記すると省略ができません。

```typescript
function distance(p1: Point, p2: Point | undefined): number
  // ...
}

distance(q1, q2);
distance(q1);
// -> Expected 2 arguments, but got 1.

distance(q1, undefined);
```

6行目のような書き方は指摘を受けます、動作させるためには9行目のように書かなければいけません。

### 省略可能な`undefined`をTypeScriptはどう解釈しているか

実はこの`Optional parameters`としても使われる省略可能な`undefined`は、TypeScriptは`void`という専用の型を作って定義しています。つまり`Optional parameters`は以下のように書き換えることもできます。

```typescript
function distance(p1: Point, p2: Point | void): number
  // ...
}

distance(q1, q2);
distance(q1);
```

この`void`型は値を指定しない`undefined`と、意図的に指定している`undefined`の2値を持っていますが`undefined`型は意図的に指定している`undefined`の1値のみを持っているため、このような差が生まれます。

### `Optional parameters`でできないこと

`Optional parameters`は必ず最後に書かなければいけません。つまり、以下のように`Optional parameters`より後ろに普通の引数を書くことはできません。

```typescript
function distance(p1?: Point, p2: Point): number {
  // ...
}
// -> A required parameter cannot follow an optional parameter.
```

### `Default parameters`

省略した時、原点との距離を求めるといったわかりやすい例であればいいのですが`(1, 2)`との距離を求める、といった変化球がきたとします。なにも考えないとこのようになります。

```typescript
function distance(p1: Point, p2?: Point): number {
  if (p2 === undefined) {
    return ((p1.x - 1) ** 2 + (p1.y - 2) ** 2) ** (1 / 2);
  }

  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}
```

もちろん動くのですが、意図がわかりにくくなってしまいます。このような時に便利なのが`Default parameters`です。`Default parameters`を使用すると以下のように書けます。

```typescript
const p0: Point = {
  x: 1,
  y: 2
};

function distance(p1: Point, p2: Point = p0): number {
  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}

distance(q1, q2);
distance(q1);
distance(q1, undefined);
```

入力がなかった時に初期値として使いたい値を、その引数の右に書きます。ここでは`p2`の右側の`= p0`がそれにあたります。

`Optional parameters`と違いユニオン型ではないため、処理の分岐が不要になります。拡張性や見通しを考えれば`Default parameters`の方に軍配が上がるでしょう。

### 初期値に関数の戻り値を使う

`Default parameters`には関数の戻り値を指定することができます。例えば、ある`(x, y)`が与えられると転置した`(y, x)`を返す`inverse()`という関数の戻り値を初期値として使用します。ちなみに`inverse()`は以下です。

```typescript
function inverse(p: Point): Point {
  return {
    x: p.y,
    y: p.x
  };
}
```

これを使うと`distance()`は以下のようになります。

```typescript
function distance(p1: Point, p2: Point = inverse(p1)): number {
  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}
```

また、`Default parameters`は`Optional parameters`と異なり、最後に書く必要はありません。呼び出し側で`Default parameters`を使用させたい時は`undefined`を指定します。この時`null`ではこの役目を果たせないので注意してください。もちろん末尾の`Default parameters`であれば省略が可能です。

```typescript
const p0: Point = {
  x: 1,
  y: 2
};

function distance(p1: Point = p0, p2: Point): number {
  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}

distance(q1, q2);
distance(undefined, q2);
distance(null, q2);
// -> Argument of type 'null' is not assignable to parameter of type 'Point | undefined'.
```

### `Default parameters`でできないこと

しかしながら、関数を`Default parameters`として使う時は非同期の関数を使うことができません。TypeScriptならびにJavaScriptは処理を非同期的に扱うことが多く、`Promise / async / await`といった非同期処理を同期的に扱うための機能があります。詳細は先の章にあるため詳しい説明は譲りますが非同期関数が値を返すまで処理を待つということはできません。

```typescript
async function distanceAync(p1: Point, p2: Point = await inverseAync(p1)): Promise<number> {
  return ((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2) ** (1 / 2);
}
```

このように`Default parameters`を書くことはできません。なお`inverseAsync()`は非同期関数とします。

## `Rest parameters`

いわゆる可変の引数のことです。たとえば引数に与えられた数値の平均を返す関数`average()`を作るとします。これを`Rest parameters`を使って表現すると以下のようになります。

```typescript
function average(...nums: number[]): number {
  if (nums.length === 0) {
    return 0;
  }

  return nums.reduce((prev: number, cur: number): number => {
    return prev + cur;
  }) / nums.length;
}

console.log(average()); // 0
console.log(average(1, 3)); // 2
```

0除算を防ぐ目的で少々処理が複雑になっていますが、可変の引数の前に`...`を付ければ、可変引数を受け付け、それらを配列として受けることができます。

### `Rest parameters`でできないこと

`Rest parameters`は最終的に配列として解釈されるからといって、引数をまとめてひとつの配列として渡すことはできません。

```typescript
average([1, 3, 5, 7, 9]);
```

このように配列を直接渡してしまうと`average()`の関数内では要素数1の`number[][]`型が渡されたと解釈されます。もちろん`average()`の期待する引数の型は`number[]`型なので、このコードを実行することはできません。

また、可変個の引数を受け付ける関係上、`Rest parameters`より後ろにほかの引数を置くことができません。ただし`Rest parameters`の前であれば問題ありません。

```typescript
function average(...nums: number[], subject: string): number {
  // ...
}
// -> A rest parameter must be last in a parameter list.
```

### `Spread syntax`

JavaScriptに組み込みのメソッドとして存在する`Math.max()`は与えられた`number`型の引数の最大値を返却しますが引数として`Rest parameters`を要求します。上記の通り配列をそのまま入れることができないので以下のようなことができません。

```typescript
const scores: number[] = mathExamination();

const max: number = Math.max(scores);
```

これは学校の試験をイメージして書いています。生徒の数は1年間ではそう増減はしないので、40人ぐらいの生徒なら力技でもなんとかなるかもしれません。

```typescript
Math.max(scores[0], scores[1], scores[2], scores[3], scores[4], scores[5], scores[6], ...);
```

書いている最中で力つきました。しかもこれは生徒が転入したり転校したりするとコードを書き換えなければならなくなります。

このような時は`Spread syntax`を使って配列を引数の列に変換します。

```typescript
const scores: number[] = mathExamination();

const max: number = Math.max(...scores);
```

`Rest parameters`も`Spread syntax`もどちらも`...`と表記しますが片方は個々の引数を配列にし、もう片方は配列を個々の引数にします。

## `Destructuring assignment`

例えばBMI\(Body Mass Index\)を計算したいとします。身長\(cm\)と体重\(kg\)が与えられれば関数`bmi()`は以下のような計算になります。

```typescript
function bmi(height: number, weight: number): number {
  const mHeight: number = height / 100.0;
  return weight / (mHeight ** 2);
}
```

この関数は引数がどちらも`number`型なので入れ替えてしまうことがあります。22は平均的な体型ですが402はややとてつもなく肥満だと言えます。

```typescript
console.log(bmi(170, 65));
// 22.49134948096886
console.log(bmi(65, 170));
// 402.36686390532543
```

このような誤用を避けるための方法として`Destructuring assignment`を使うことができます。`Destructuring assignment`を使うと以下のように書きなおせます。

```typescript
type TopSecret = {
  height: number;
  weight: number;
};

function bmi({height, weight}: TopSecret): number {
  const mHeight: number = height / 100.0;
  return weight / (mHeight ** 2);
}
```

呼び出しは以下のようになります。これなら`height`と`weight`の意味を取り違えない限り問題は起こりにくくなるでしょう。以下は同じ結果を返します。

```typescript
bmi({height: 170, weight: 65});
bmi({weight: 65, height: 170});
```

すでに`height, weight`という変数が定義済みであればこのように書くこともできます。

```typescript
const height: number = 170;
const weight: number = 65;

bmi({height, weight});
bmi({weight, height});
```

### `Destructuring assignment`でうれしいこと

`Destructuring assignment`は普通の引数と異なり以下のような利点があります。

#### 引数の順番にとらわれない

これは、上記の通りです。

#### `Default parameters`と併用できる

身長あるいは体重を省略できるようにして、省略時に初期値を入れるようにすることができます。

```typescript
function bmi({height = 165, weight = 60}: Partial<TopSecret>): number {
  const mHeight: number = height / 100.0;
  return weight / (mHeight ** 2);
}
```

なお、`Partial<T>`とは、オブジェクト`T`のプロパティ、メソッドを省略可能にします。つまり`Partial<TopSecret>`は以下と同じです。この時の`?`は引数で説明した`Optional parameters`と意味するものは同じです。

```typescript
type PartialTopSecret = {
  height?: number;
  weight?: number;
};
```

これによって呼び出し側は`bmi()`を以下のどのような方法でも呼び出すことができます。

```typescript
bmi({});
bmi({height: 180});
bmi({weight: 75});
bmi({height: 180, weight: 75});
bmi({weight: 75, height: 180});
```

さらに以下のように引数の型の右にも`Default parameters`を付けてあげれば引数自体を省略することができるようになります。

```typescript
function bmi({height = 165, weight = 60}: Partial<TopSecret> = {}): number {
  // ...
}

bmi();
```

## 引数の`this`

`function`とクラスのメソッドの第1引数は`this`という特殊な引数を受けることができます。これは使用するコンテキストによって`this`の意味するところが変わってしまうこれらがどのコンテキストで使用されるべきなのかをTypeScriptに伝えるために使います。この`this`は呼び出し側では引数として指定する必要はなく、第2引数以降を指定すれば問題ありません。

なお、アロー関数はこの`this`を持つことができません。それは前述のとおりアロー関数は宣言時の`this`が使われるため`this`が変動することがなく、そもそも不要だからです。

```typescript
class Male {
  private name: string;
  
  public constructor(name: string) {
    this.name = name;
  }
  
  public toString(): string {
    return `Monsieur ${this.name}`;
  }
}

class Female {
  private name: string;
  
  public constructor(name: string) {
    this.name = name;
  }
  
  public toString(this: Female): string {
    return `Madame ${this.name}`;
  }
}
```

上記クラス`Male, Female`はほぼ同じ構造ですが`toString()`のメソッドが異なります。

`Male, Female`はともに普通の用途で使うことができます。

```typescript
const male: Male = new Male('Frédéric');
const female: Female = new Female('Frédérique');

console.log(male.toString());
// -> Monsieur Frédéric
console.log(female.toString());
// -> Madame Frédérique
```

ですが、各インスタンスの`toString`を変数に代入すると意味が変わります。

```typescript
const maleToStr: () => string = male.toString;
const femaleToStr: (this: Female) => string = female.toString;

console.log(maleToStr());
conosle.log(femaleToStr());
// -> The 'this' context of type 'void' is not assignable to method's 'this' of type 'Female'.
```

`femaleToStr()`のコンテキストが`Female`ではないとの指摘です。このコードを実行することはできません。  
ちなみにこの対応をしていない`maleToStr()`は実行こそできますが実行時に例外が発生します。

```typescript
    return `Monsieur ${this.name}`;
                            ^
TypeError: Cannot read property 'name' of undefined
```

## 戻り値 `(Return value)`

関数の戻り値を指定することができます。あえて書かなくてもTypeScript側で補完し、関数の戻り値として提供してくれますが、意図しない戻り値を返していないかの検査が働くので書いた方が良いでしょう。前述の通り書く位置が実装と型で異なることに注意してください。

戻り値はJavaScriptと同じく1値のみの返却です。一度に多くの値を戻したい場合はタプルの章を参照してください。

戻り値がないことを明示したい時は`void`と書きます。内部では`undefined`を返していることと同義です。ただし、戻り値を`undefined`と明記した時と違う点があります。それは`return`のない関数は`void`である必要があります。  
以下の例では`doNothing4()`は`return`のない関数で戻り値の型が`undefined`なのでTypeScriptから指摘を受けます。それ以外は問題がありません。

```typescript
function doNothing1(): void {
}

function doNothing2(): void {
  return;
}

function doNothing3(): void {
  return undefined;
}

function doNothing4(): undefined {
}
// -> A function whose declared type is neither 'void' nor 'any' must return a value.

function doNothing5(): undefined {
  return;
}

function doNothing6(): undefined {
  return undefined;
}
```

これは`undefined`型が明示的な`undefined`の1値のみを持つのに対し`void`型は明示的な`undefined`と暗黙の`undefined`の2値を持つことに由来します。

```typescript
function returnUnfefined(): undefined {
  return undefined;
}

function returnVoid(): void {
}

const u1: undefined = returnUnfefined();
const u2: void = returnUnfefined();

const v1: undefined = returnVoid();
// -> Type 'void' is not assignable to type 'undefined'.
const v2: void = returnVoid();
```

## `Type predicate`

プログラムを書いているとその変数が意図する型なのかをはっきりさせたい時があります。

```typescript
function isDuck(animal: Animal): boolean {
  if (walksLikeDuck(animal)) {
    if (quacksLikeDuck(animal)) {
      return true;
    }
  }
  
  return false;
}
```

動物がアヒルかどうかを判定するその名も`isDuck()`です。

ですが、この関数は使用者に対してその変数がアヒルかどうかを伝えているだけです。TypeScriptに対してそれがアヒルであることを伝えるためにはキャストが必要になります。

```typescript
if (isDuck(animal)) {
  const duck: Duck = animal as Duck;
  duck.quacks();
  // ...
}
```

`as`はTypeScriptにおけるキャストの一種です。名前を`Type Assertions`と言いますが他の言語と異なりかなり強引な型変換ができてしまいます。使用には気をつけてください。ちなみに別の表記方法があるのですが現在はあまり推奨されていません。理由は`JSX`のタグと見分けがつかないからです。

```typescript
const duck1: Duck = animal as Duck;
const duck2: Duck = <Duck> animal;
```

これをより賢くやろうというのが`Type predicate`です。

### `Type predicate`の宣言

`Type predicate`の宣言は戻り値が`boolean`型の関数に対して適用でき、戻り値の`boolean`を以下のように書き替えます。

```typescript
function isDuck(animal: Animal): animal is Duck {
  // ...
}
```

これで関数`isDuck()`が`true`を返す時の`if`のブロックの中では`animal`は`Duck`型として解釈されるようになります。

```typescript
if (isDuck(animal)) {
  animal.quacks();
  // ...
}
```

## `Assertion functions`

やりたいことはほぼ`Type predicate`と同じです。`Type predicate`は`boolean`型の戻り値に対して使用しましたがこちらは例外を投げるかどうかで判定します。上記関数`isDuck()`を`Assertion functions`で書きかえると以下のようになります。

```typescript
function isDuck(animal: Animal): asserts animal is Duck {
  if (walksLikeDuck(animal)) {
    if (quacksLikeDuck(animal)) {
      return;
    }
  }
  
  throw new Error('YOU ARE A FROG!!!');
}

// ...

isDuck(animal);

animal.quacks();
```

こちらはこの関数が呼ばれた後であればいつでも変数`animal`は`Duck`型として解釈されます。

## `Type predicate, Assertion functions`のつかいかた

値が存在するかしないかを表現する時、言語によっては`Optional`という入れ物のクラスを用意することがあります。このクラスを抽象クラスとして定義し、サブクラスに値が存在する`Some`と存在しない`None`を用意すると`Optional`に`Type predicate`を使うことができます。

```typescript
abstract class Optional<T> {
  // ...
  public abstract isPresent(): this is Some<T>;
}

class Some<T> extends Optional<T> {
  // ...
  public isPresent(): this is Some<T> {
    return true;
  }
}

class None<T> extends Optional<T> {
  // ...
  public isPresent(): this is Some<T> {
    return false;
  }
}
```

### `Type predicate`で注意すること

上記`Optional`の例が顕著なのですが、`optional.isPresent()`が`false`を返したからと言ってTypeScriptは変数`optional`が`None`であるとは解釈しません。あくまでも`Some`ではないと解釈されるだけです。

```typescript
if (optional.isPresent()) {
  // optional is Some<T>
} else {
  // optional is something else but Some<T>
}
```

また`Type predicate`は`false`の場合を定義することができません。つまり以下のような定義はできません。

```typescript
public abstract isPresent(): this is Some<T>, this is not None<T>;
```

このような時は専用のメソッドを用意します。

```typescript
abstract class Optional<T> {
  // ...
  public abstract isPresent(): this is Some<T>;
  
  public abstract isAbsent(): this is None<T>;
}
```

ただこの例の場合TypeScriptではユニオン型によって簡単に解決できます。ユニオン型については詳細の説明があるのでそちらをご参照ください。

## 戻り値の`this`

四則演算ができる変哲もないクラス`Operator`を考えます

```typescript
class Operator {
  protected value: number;

  public constructor(value: number) {
    this.value = value;
  }
  
  public sum(value: number): void {
    this.value += value;
  }

  public subtract(value: number): void {
    this.value -= value;
  }

  public multiply(value: number): void {
    this.value *= value;
  }

  public devide(value: number): void {
    this.value /= value;
  }
}

const op: Operator = new Operator(0);

op.sum(5); // 5
op.subtract(3); // 2
op.multiply(6); // 12
op.devide(3); // 4
```

演算ごとにステートメントを切る必要があります。  
このような場合メソッドチェインを使って処理を連続させることができます。

```typescript
class Operator {
  protected value: number;

  public constructor(value: number) {
    this.value = value;
  }

  public sum(value: number): Operator {
    this.value += value;
    return this;
  }

  public subtract(value: number): Operator {
    this.value -= value;
    return this;
  }

  public multiply(value: number): Operator {
    this.value *= value;
    return this;
  }

  public devide(value: number): Operator {
    this.value /= value;
    return this;
  }
}

const op: Operator = new Operator(0);
op.sum(5).subtract(3).multiply(6).devide(3); // 4
```

`op.sum(), op.subtract(), op.multiply(). op.devide()`の戻り値の型を`Operator`に変更しました。これによりメソッドチェインが可能になりました。

ここで、このクラス`Operator`を拡張して累乗の計算を追加したいとします。すると新しいクラス`NewOperator`は以下のようになるでしょう。

```typescript
class NewOperator extends Operator {

  public constructor(value: number) {
    super(value);
  }

  public power(value: number): NewOperator {
    this.value **= value;
    return this;
  }
}
```

ですが、このクラスでは以下の演算ができません。

```typescript
const op: NewOperator = new NewOperator(2);
op.power(3).multiply(2).power(3);
// -> Property 'power' does not exist on type 'Operator'.
```

これは`op.multiply()`の戻り値が`Operator`だからです。`Operator`には`power()`というメソッドがないためこのような問題が発生します。

このような時、戻り値に`this`を設定することができます。上記クラスの戻り値の`Operator, NewOperator`を全て`this`に置き換えると問題が解消されます。

```typescript
class Operator {
  protected value: number;

  public constructor(value: number) {
    this.value = value;
  }

  public sum(value: number): this {
    this.value += value;
    return this;
  }

  public subtract(value: number): this {
    this.value -= value;
    return this;
  }

  public multiply(value: number): this {
    this.value *= value;
    return this;
  }

  public devide(value: number): this {
    this.value /= value;
    return this;
  }
}

class NewOperator extends Operator {

  public constructor(value: number) {
    super(value);
  }

  public power(value: number): this {
    this.value **= value;
    return this;
  }
}

const op: NewOperator = new NewOperator(2);
op.power(3).multiply(2).power(3); // 4096
```

## オーバーロード `(Overloads)`

オーバーロードとは、関数の名称は同じでありながら異なる引数、戻り値を持つことができる機能です。TypeScriptもこの機能を用意しているのですが、大元がJavaScriptであることが災いし、やや使いづらいです。

### オーバーロードの定義

オーバーロードはその関数が受け付けたい引数、戻り値の組を実装する関数の上に書きます。例えば先ほど使用した2点の距離を求める関数`distance()`をオーバーロードで定義すると以下のようになります。なお、この例では戻り値は全て`number`型ですが、別の型にしても問題ありません。

```typescript
function distance(p: Point): number;
function distance(p1: Point, p2: Point): number;
function distance(x: number, y: number): number;
function distance(x1: number, y1: numebr, x2: number, y2: number): number;
```

なお、上記のような書き方のオーバーロードは名前付き関数`(Normal functions)`またはクラスのメソッドでのみ可能です。匿名関数`(Anonymous functions)`、アロー関数`(Arrow functions)`では、オーバーロードを定義したタイプエイリアスまたはインターフェイスを定義します。たとえば、上記例だと以下のようなタイプエイリアスになります。

```typescript
type Distance = {
  (p: Point): number;
  (p1: Point, p2: Point): number;
  (x: number, y: number): number;
  (x1: number, y1: number, x2: number, y2: number): number;
};

const distance: Distance = (arg1: number | Point, arg2?: number | Point, arg3?: number, arg4?: number): number => {
  // ...
};
```

### オーバーロードの実装

ここからが大変です。実装はオーバーロードで定義した全てをひとつの関数で処理しなければいけません。つまり`distance()`の実装は以下のようになります。これが呼び出し側では**あたかも**他言語のオーバーロードのようになります。

```typescript
function distance(p: Point): number;
function distance(p1: Point, p2: Point): number;
function distance(x: number, y: number): number;
function distance(x1: number, y1: numebr, x2: number, y2: number): number;
function distance(
  arg1: Point | number,
  arg2?: Point | number,
  arg3?: number,
  arg4?: number
): number {
  // ...
}

distance(q1);
distance(q1, q2);
distance(1, 3);
distance(1, 3, 5, 7);
```

### オーバーロードでうれしいこと

オーバーロードを定義しないで実装する、つまりオーバーロードの定義なしに実装すると以下のような引数を考慮しなければなりません。

```typescript
distance(q1, 5, undefined, 8);
```

オーバーロードを定義しておくことで意図する引数と戻り値の組み合わせを定義できるようになります。上記引数はオーバーロードの定義によりTypeScriptから指摘を受けます。

```typescript
Argument of type 'Point' is not assignable to parameter of type 'number'.
```

これはTypeScriptが`distance()`を`number`型の引数4個版で受けていると解釈している時の指摘です。


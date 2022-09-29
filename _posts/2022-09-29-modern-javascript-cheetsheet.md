---
title: モダンJavaScriptの簡単まとめ
category: "Javascript"
tags: [JavaScript, React]
---

UdemyでReactコースを勉強し始めて、最初からReactでよく使うJavascriptの新機能について紹介があったので、Javascriptの復習としてメモする。

### 変数宣言 `var` / `const` / `let`

結論：基本的に`const`を使う。`var`を使わない。

一度宣言した変数を後から再代入する必要がある場合は、`let`を使う。

例：
```javascript
const name = 'Mike';
name = 'Nick' // エラーが起こる。constで宣言する変数は再代入不可。

let name = 'Mike';
name = 'Nick' // letで宣言する変数は再代入可能。
```

### アロー関数

通常の`function`関数宣言構文より、アロー関数のメリットは、
- 構文がより簡潔
- `this`のスコープは周りと一致する
- 暗黙的な `return`

例：
```javascript
// 従来の書き方
function callMe(name) { 
    console.log(name);
}

//アロー関数
const callMe = (name) => { 
    console.log(name);
}

// 引数が一個のみの場合、引数の括弧が省略できる
const callMe = name => { 
    console.log(name);
}

// 単純に値を返す場合、returnを省略できる
  const double = x => x * 2;

// returnを明記する書き方
 const double = (x) => {
    return x * 2; 
  }
```

### Exports & Imports

**2種類のexports: **  

- `default exports`：  

モジュールごとにひとつだけdefault exportが指定できる

```javascript
const person = {
  name: 'Max'
}

export default person 
```
{: file="person.js" }

他のファイルでimportする時、任意の名前でimportできる

```javascript
// 元の名前でimport
import person from './person.js'
// 任意の名前でimport、例えば people
import people from './person.js'
```
{: file="app.js" }


- `named exports`(名前付きエクスポート): 

```javascript
export const clean = () => {...}
export const baseData = 10;
```
{: file="utility.js" }

他のファイルでimportする時、名前通りにimportする必要

```javascript
import { clean, baseData } from './utility.js'

// 新しい名前でエイリアス設定可能
import { baseData as data } from './utility.js'

// まとめてimport可能
import * as utilities from './utility.js'
consoloe.log(utilities.baseData) // 10
```
{: file="app.js" }


### Classes

オブジェクト指向の言語でのclassと似ている。  
`extends`でクラスの継承もできる。

```javascript
class Human {
  constructor() {
    this.age = 30; // `this`はrubyやpythonの中の`self`のような感じ？
  }
  printMyAge() {
    console.log(this.age)
  }
}

class Person extends Human {
  constructor() {
    super(); // 親クラスのconstructorが呼ばれる
    this.name = 'Max';
    this.age = 20; //属性のオーバライドできる
  }

  printMyName() {
    console.log(this.name)
  }
}

const person = new Person();
person.printMyName; //  Max
person.printMyAge; // 20
```

ES7以降の仕様は`constructor`が不要になるので、もっと簡潔にできる
```javascript
class Human {
  age = 20;
}
 
class Person extends Human {
  name = 'Max';

  printMyName = () => {
    console.log(this.name);
  }
}
 
const person = new Person();
person.printMyName(); // Max
console.log(person.age); // 20
```

### Spread / Rest Operator "`...`" 

`...`演算子を使って、配列やオブジェクトなどを簡単に分割/複製できる

```javascript
const numbers = [1, 2, 3];
const newNumbers = [...numbers, 4, 5]; 

console.log(newNumbers); // [1, 2, 3, 4, 5]

const person = {
  name: 'Max'
};

const newPerson = {
  ...person,
  age: 20
}

/* newPersonの中身 =>
{
  name: 'Max',
  age: 28
}
*/
```
`...`はレスト演算子として、関数の引数をまとめて配列に入れ込むことができる。

```javascript
const filter = (...args) => {
  return args.filter(number => number >= 2);
}

console.log(filter(1, 2, 3)) // [2, 3]
```

### オブジェクトや配列の分割(Destructuring)

```javascript
// 配列を分割する
const array = [1, 2, 3];
const [num1, num2] = array;
console.log(num1, num2); // 1 2

// 中に空のスペースを入れると
const [num1, , num3] = [1, 2, 3];
console.log(num1, num3); // 1 3


// オブジェクトを分割する
const myObj = {
  name: 'Max',
  age: 28
}

const {name} = myObj;
console.log(name); // Max
console.log(age); // undefined
console.log(myObj); // {name: 'Max', age: 28}
```

### Primitive / Reference Data Types

基本型と参照型は新しい概念ではないけど、よく忘れる重要な概念なので、一旦メモする。

基本型データ：numbers, strings, booleans, null, undefinedなど
参照型データ：Arrays, Objects, Functions, Collections, Datesなど

この二つのデータ型の違いはデータの保存形式になる。

基本型例：
```javascript
let number = 1; 
let num2 = number;

number = 2
console.log(number) // 2
console.log(num2) // 1
```
`const num2 = number`は変数`number`の値「1」をコピーして変数`num2`に渡す。

変数`number`と変数`num2`はそれぞれの独立の値を持ち、メモリ空間でそれぞれの値を格納する。

そのため、`number`の値を変更しても、`num2`には影響がない。

参照型例：
```javascript
const person = {
  name: 'Max'
};

const secondPerson = person; 

secondPerson.name = 'John'
console.log(person.name); // John
```
`const person = {..}`で`person`は独立のオブジェクトを持つのではなく、
メモリに格納するのは`{ name: 'Max' }`というオブジェクトだけ。
変数`person`が保存するのは`{ name: 'Max' }`オブジェクトが格納された場所を指すポインター。  
ポインターというのは、そのオブジェクトが格納された場所(メモリのアドレス)を保存するもの。

`const secondPerson = pserson`で、新しい`{ name: 'Max' }`オブジェクトを作るのではなく、先作ったオブジェクトのアドレスをコピしてポインターとして`secondPerson`に渡すだけ。

そのため、`secondPerson.name`を変更したら、`secondPerson`が指すオブジェクト`{ name: 'Max' }`自体を`{ name: 'John' }`に変更することになるので、
同じオブジェクトを指す`person`の中身も変更になる。

![画像](https://i.imgur.com/FcVrHmy.png)

**`...`演算子を使うと、オブジェクトをコピできる**

通常の参照型データと違って、`...`演算子を使うと、オブジェクトそのものをコピーできる
```javascript
const person = {
  name: 'Max'
};

const secondPerson = {
  ...person
}

person.name = 'John';
console.log(secondPerson.name); // 'Max'
```
ここで`secondPerson`が持つのは、ポインターではなく、{ name: 'Max' }オブジェクト自体をコピーして、独立したオブジェクト。

そのため、`person.name = 'John'`で元のオブジェクトを変更しても、コピしたオブジェクトには影響はない。

### 配列でよく使う関数 `map()`, `filter()`, `reduce()`

**`map()`**

配列内の要素を全部呼び出して、その結果を新しい配列として生成する。

```javascript
const array1 = [1, 4, 9, 16];
const map1 = array1.map(x => x * 2);

console.log(map1); // [2, 8, 18, 32]
```

**`filter()`**

条件を満たす要素を抽出して新しい配列を生成する。

```javascript
const nums = [1,2,3,4,5,6]

console.log(nums.filter(num => num > 3)) // [4, 5, 6]
```

**`reduce()`**

配列内に対して関数を適用し、その処理によって配列の要素を(左から右に)一つの値にまとめる

```javascript
const nums = [1,2,3]

let initValue = 0;
const sumNumbers = nums.reduce(
  (prev, curr) => prev + curr, initValue
);

console.log(sumNumbers); // 6 <= 0+1+2+3

initValue = 2;

const multiplyNumbers = nums.reduce(
  (prev, curr) => prev * curr, initValue
)

console.log(multiplyNumbers); // 12 <= 2*1*2*3
```

---
参照  
[React - The Complete Guide (incl Hooks, React Router, Redux)](https://www.udemy.com/course/react-the-complete-guide-incl-redux)

[モダン JavaScript チートシート](https://mbeaudru.github.io/modern-js-cheatsheet/translations/ja-JP.html)

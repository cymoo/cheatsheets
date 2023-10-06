# TypeScript cheatsheet

## 基础

* `any` (top and bottom type), `unknown` (top type), `never` (bottom type)

* 基本类型：`boolean`, `string`, `number`, `bigint`, `symbol`, `object`, `undefined`, `null`
  
* 包装类型：`Boolean`, `String`, `Number`，不建议使用包装类型

  ```typescript
  const s1:String = 'hello'; // 正确
  const s2:String = new String('hello'); // 正确
  const s3:string = 'hello'; // 正确
  const s4:string = new String('hello'); // 报错
  ```

* `Object` and `object`

  * `Object`：广义的对象，除了`undefined`和`null`这两个值不能转为对象，其他任何值都可以赋值给`Object`类型，可简写为`{}`
  * `object`：狭义对象，只包括对象、函数和数组，不包括原始类型

* `undefined` and `null`

  * `undefined` 和 `null` 既可以作为值，也可以作为类型，取决于在哪里使用它们；没有声明类型的变量，被赋值为`undefined`或`null`，在关闭编译设置`noImplicitAny`和`strictNullChecks`时，它们的类型会被推断为any。

  * 任何其他类型的变量都可以赋值为`undefined`和`null`

  ```typescript
  let age: number = 24;

  age = null;      // 正确
  age = undefined; // 正确
  ```

  * 但是如果打开编译选项`strictNullChecks`，它们就不能赋值给其他类型变量（除了`any`和`unknown`），打开此选项后，它们也不能相互赋值。

* 数组

  ```typescript
  let arr1: number[] = [1, 2, 3]
  let arr2: (number|string)[] = [1, 2, 3]
  let arr3: Array<number> = [1, 2, 3]

  // 可使用方括号读取数组成员类型
  type Names = String[]
  type Name1 = Names[0] // string
  // or
  type Name2 = Names[number] // string

  // 只读数组，是普通数组的父类型
  const arr4: readonly number[] = [0, 1]
  const arr5: ReadonlyArray<number> = [0, 1]
  // 还可以使用`const断言`来声明
  const arr6 = [0, 1] as const

  // 多维数组
  var multi: number[][] = [[1, 2], [3, 4]]
  ```

* 元组

  ```typescript
  const s: [string, string, boolean] = ['a', 'b', true]

  // 成员可选，问号只能用于尾部，即可选成员在最后
  let a: [number, number?] = [1]

  // 使用扩展运算符，可以不限成员数量
  type NamedNums = [string, ...number[]]
  const b: NamedNums = ['A', 1, 2]
  const c: NamedNums = ['B', 1, 2, 3];
  // 扩展运算符可以用在任意位置
  type T1 = [string, number, ...boolean[]]
  type T2 = [string, ...boolean[], number]
  type T3 = [...boolean[], string, number]

  // 元组可以添加成员名，它只是说明性的，没有实际作用
  type Color = [red: number, green: number, blue: number]
  const c: Color = [255, 255, 255]

  // 可通过方括号，读取成员类型
  type Tuple = [string, number]
  type Age = Tuple[1] // number
  type Foo = Tuple[number] // string | number

  // 只读元组，是普通元组的父类型
  type T4 = readonly [number, string]
  // 或者使用工具类型
  type T5 = Readonly<[number, string]>
  ```

* symbol

  ```typescript
  let s: symbol = Symbol()
  
  // 报错
  const x: symbol = Symbol();
  // 正确, unique symbol是symbol的子类型
  const y: unique symbol = Symbol();
  
  // 如果把symbol当做属性名，类型只能是unique symbol
  const x:unique symbol = Symbol();
  const y:symbol = Symbol();
  interface Foo {
    [x]: string; // 正确
    [y]: string; // 报错
  ```
  
* 值类型

  ```typescript
  let x: 'hello'
  const x = 'https'
  ```

* never
  用在返回返回值中，表示该函数不会正常执行结束

  ```typescript
  // 抛出错误的函数
  function fail(): never {
    throw new Error("sorry")
  }
  
  // 无限执行的函数
  function runForever(): never {
    while (true) {}
  }
  ```

* 联合类型
  
  ```typescript
  let x: string | number
  let gender: 'male' | 'female'
  ```

* 交叉类型
  
  ```typescript
  let x: number & string // never
  
  let obj:
    { foo: string } &
    { bar: string };
  
  obj = {
    foo: 'hello',
    bar: 'world'
  };
  
  type A = { foo: number };
  
  type B = A & { bar: number };
  ```

* type命令：用来定义别名，`type Age = number`

* typeof运算符：TypeScript将其移植到了类型运算，操作数仍然是一个值（只能是标识符），但是返回的不是字符串，而是该值的TypeScript类型

    ```typescript
    const a = { x: 0 };
    
    type T0 = typeof a;   // { x: number }
    type T1 = typeof a.x; // number
    ```

## 函数

```typescript
// 普通函数
function foo(txt: string): void {
  console.log('hello ' + txt)
}

// 表达式
const bar: (txt: string) => void = function(txt) {
  console.log('hello ' + txt)
}

// arrow function
const baz = (x: number, y: number): number => x + y 

// 实际参数可以少于类型指定的参数
let func: (a: number, b: number) => number
func = (a: number) => a  // OK

// 函数类型可以采用对象写法，较少使用，适用于函数本身存在属性的情况
let add: {
  (x: number, y: number): number
  version: string
}
add = function(x, y) {return x + y}
add.version = '1.0'

// 可选参数
function f(x?: number) {}
function f1(x: number | undefined) {}
f1() // Error：x不能省略

// 参数解构
function f([x, y]: [number, number]) {}
function f({a, b, c}: {a: number, b: number, c: number}) {}

// rest参数
// rest为数组
function f(...nums: number[]) {}
// rest为元组
function f(...nums: [string, number]) {}
function f(...nums: [string, number?]) {}
// rest可以嵌套
function f(x: number, ...nums: [string, ...number[]]) {}

// readonly
function sumOfArray(arr: readonly number[]) {
  // ...
  arr[0] = 0 // Error
}

// void
// 对于函数字面量，如果返回值是void类型，则不能有返回值
function f(): void {
  // return 123 // Error
  // return null // OK when strictNullChecks = false
  // return undefined // OK
}
// NOTE: 但是对于变量，函数参数等，可以接受返回任意值的函数
type voidFunc = () => void
const f: voidFunc = () => {return 123}
// 这里的void只是表示该函数的返回值没有利用价值，或是不需要使用；
// 例如Array.prototype.forEach(fn)的fn返回值为void，但实际传入的函数是有返回值的；

// never
// 如果函数在某些情况下有正常返回值，其他情况抛出错误，则返回值可以省略never
function sometimesThrow(): number {
  if (Math.random() > 0.5) return 100
  throw new Error('oh no')
}
// 类型推断为number，而不是number|never；
// 原因为never为bottom type，number|never = number
// 无论返回函数值是什么类型，都可能包含了抛出错误的情况
const result = someTimesThrow()

// 函数重载
function reverse(str: string): string;
function reverse(arr: any[]): any[];
function reverse(stringOrArray: string|any[]): string|any[] {
  if (typeof stringOrArray === 'string')
    return stringOrArray.split('').reverse().join('');
  else
    return stringOrArray.slice().reverse();
}
// 函数重载也可以用对象表示
type CreateElement = {
  (tag:'a'): HTMLAnchorElement;
  (tag:'canvas'): HTMLCanvasElement;
  (tag:'table'): HTMLTableElement;
  (tag:string): HTMLElement;
}
function createElement(tag: string): HTMLElement {}

// 构造函数
// 参数列表前加上new
class Animal {
  numLegs:number = 4;
}
type AnimalConstructor = new () => Animal;

function create(c:AnimalConstructor):Animal {
  return new c();
}
const a = create(Animal);
// 构造函数还可以采用对象形式
type F = {
  new (s:string): object;
};
// 某些函数既可以当做普通函数，也可以作为构造函数
type F = {
  new (s:string): object;
  (n?:number): number;
}
```

## 对象

```typescript
const obj:{
  x: number;  // 也可用逗号,
  y: number;
  readonly z: number;
  add(x:number, y:number): number;
  // 或者写成
  // add: (x:number, y:number) => number;
} = {
  x: 1,
  y: 1,
  z: 1,
  add(x, y) {
    return x + y;
  }
};

// 读取属性类型
type User = {name: string, age: number}
type Name = User['name']  // string

// 可选属性
// 打开ExactOptionalPropertyTypes和strictNullChecks编译选项，可选属性就不能设为undefined
type User = {firstName: string, lastName?: string}

// 只读
const user = {name: 'foo'} as const

// 属性名的索引
type MyObj = {
  // property只有三种可能的类型：string, number, symbol 
  [property: string]: string
}
const obj: MyObj = {
  foo: 'a',
  bar: 'b',
  baz: 'c',
}
// 可以同时有数值索引，但是不能与字符串冲突，因为在js里，所有的数值属性都会转为字符串
type MyType = {
  [x: number]: boolean; // 报错
  // 也可以同时声明耽搁属性，但不能与索引冲突
  foo: boolean; // 报错
  [x: number]: string;
  [x: string]: string;
}

// 解构，与对象声明一样
const {id, name, price}: {
  id: string;
  name: string;
  price: number
} = product

// 结构类型原则
// B可以赋值给A，因为含有x属性，即子类型兼容父类型
type A = {x: number}
type B = {x: number; y: number}
const b: B = {x: 1, y: 1}
const a: A = b; // 正确
// NOTE: 但可能发生奇怪的结果
// 因为所有与myObj兼容的对象都可以传入，即obj[n]取出的不一定是数值；
// 因为v被推断为any
type myObj = {x: number, y: number}
function getSum(obj: myObj) {
  let sum = 0;
  for (const n of Object.keys(obj)) {
    const v = obj[n]; // 报错
    sum += Math.abs(v);
  }
  return sum;
}

// 严格字面量检查
// 如果对象使用字面量表示，字面量结构与类型不一致，就会报错
// 如果等号右边是一个变量，根据结构类型原则，不会报错
const point: {
  x:number;
  y:number;
} = {
  x: 1,
  y: 1,
  z: 1 // 报错，suppressExcessPropertyErrors可以关闭检查此错误
};

// 最小可选属性规则
// 如果某个类型的所有属性都是可选的，该类型的对象必须至少存在一个可选属性，不能所有可选属性都不存在
type Options = {
  a?:number;
  b?:number;
  c?:number;
}
const opts = { d: 123 };
const obj:Options = opts; // 报错

// 空对象
const obj = {};  // ts推断obj为空对象，即 const obj: {} = {}
obj.prop = 123; // 报错
const pt = {};
pt.x = 3;  // Error
pt.y = 4;  // Error
// 正确
const pt = {x: 3, y: 4};
// 如果需要分步声明，使用扩展运算符
const pt0 = {};
const pt1 = { x: 3 };
const pt2 = { y: 4 };
const pt = {...pt0, ...pt1, ...pt2};
// 空对象不会有严格字面量检查，复制可以允许多余的属性，但是不能读
interface Empty { }
const b:Empty = {myProp: 1, anotherProp: 2}; // 正确
b.myProp // 报错
// 强制使用没有属性的空对象
interface WithoutProperties {[key: string]: never}
const a:WithoutProperties = { prop: 1 };  // Error
```

## 接口

* Interface的成员有5形式：

  ```typescript
  // 对象属性与索引属性
  interface Point {
    x: number;
    y?: number;
    readonly z: number;
    [prop: string]: number;
  }

  // 对象方法，有3种写法
  // 写法一
  interface A {
    f(x: boolean): string;
  }
  // 写法二
  interface B {
    f: (x: boolean) => string;
  }
  // 写法三
  interface C {
    f: { (x: boolean): string };
  }
  // 表达式也可以
  const f = 'f'
  interface A {
    [f](x: boolean): string;
  }
  // 方法重载
  interface A {
    f(): number;
    f(x: boolean): boolean;
    f(x: string, y: string): string;
  }
  function myFunc(): number;
  function myFunc(x: boolean): boolean;
  function myFunc(x: string, y: string): string;
  function myFunc(
    x?:boolean|string, y?:string
  ):number|boolean|string {
      // ...
  }
  const a: A = {f: myFunc}

  // 函数
  interface Add {
    (x:number, y:number): number;
  }
  const myAdd:Add = (x,y) => x + y;

  // 构造函数
  interface ErrorConstructor {
    new (message?: string): Error;
  }
  ```

* interface可以继承其他类型

  ```typescript
  // 继承其他interface（可多继承），若与父接口有同名属性，则必须是类型兼容的
  interface Foo {
    id: string;
  }
  interface Bar extends Foo {
    id: number; // 报错
  }

  // 继承type
  type Country = {
    name: string;
    capital: string;
  }
  interface CountryWithPop extends Country {
    population: number;
  }

  // 继承class
  class A {
    x:string = '';

    y():boolean {
      return true;
    }
  }
  interface B extends A {
    z: number
  }
  // 某些类拥有私有成员和保护成员，interface 可以继承这样的类，但没有意义
  // 如下，B继承了A，但因为有私有和保护成员，则无法用于对象类型
  // 此时B只用被其他class实现，但其他class与A不构成父子类关系，所以x与y无法实现
  class A {
    private x: string = '';
    protected y: string = '';
  }
  interface B extends A {
    z: number
  }
  // 报错
  const b: B = { /* ... */ }
  // 报错
  class C implements B {
    // ...
  }
  ```

* interface合并

  ```typescript
  // 同名接口合并为一个
  interface Box {
    height: number;
    width: number;
  }
  interface Box {
    length: number;
  }
  
  // 主要用于兼容js行为
  interface Document {
    foo: string;
  }
  document.foo = 'hello';
  
  // 接口合并时，同一个属性如果有多个类型声明，则彼此不能冲突
  interface A {
    a: number;
  }
  interface A {
    a: string; // 报错
  }
  
  // 同名接口合并时，如果同名方法有不同的类型声明，那么会发生函数重载。
  // 而且，后面的定义比前面的定义具有更高的优先级。
  interface Cloner {
    clone(animal: Animal): Animal;
  }
  interface Cloner {
    clone(animal: Sheep): Sheep;
  }
  interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
  }
  // 等同于
  interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
  }
  // NOTE：有个例外，同名方法之中，如果有一个参数是字面量类型，字面量类型有更高的优先级。
  interface A {
    f(x:'foo'): boolean;
  }
  interface A {
    f(x:any): void;
  }
  // 等同于
  interface A {
    f(x:'foo'): boolean;
    f(x:any): void;
  }
  
  // 如果两个 interface 组成的联合类型存在同名属性，那么该属性的类型也是联合类型
  interface Circle {
    area: bigint;
  }
  interface Rectangle {
    area: number;
  }
  declare const s: Circle | Rectangle;
  s.area;   // bigint | number
  ```

* interface与type的区别

  它们很多时候可以相互替代，区别主要有以下

  * type可以表示非对象类型

  * interface可以继承，如果type定义的想添加属性，只能使用&运算符，type也可以&interface

  * 多个interface可以合并，而type则报错

  * interface不能包括属性映射（mapping），type不行

    ```typescript
    interface Point {
      x: number;
      y: number;
    }
    
    // 正确
    type PointCopy1 = {
      [Key in keyof Point]: Point[Key];
    };
    // 报错
    interface PointCopy2 {
      [Key in keyof Point]: Point[Key];
    };
    ```

  * `this`只能用于interface

    ```typescript
    // 正确
    interface Foo {
      add(num:number): this;
    };
    // 报错
    type Foo = {
      add(num:number): this;
    };
    ```

  * type可以扩展原始类型

    ```typescript
    // 正确
    type MyStr = string & {
      type: 'new'
    };
    // 报错
    interface MyStr extends string {
      type: 'new'
    }
    ```

  ## class

  * 属性声明

    可在顶层声明，也可在构造方法内部声明

    ```typescript
    // 打开 strictPropertyInitialization
    class Point {
      x: number; // 报错
      y: number; // 报错
    }
    // 打开以上选项后，使用非空断言则不报错
    class Point {
      x!: number;
      y!: number;
    }
    
    
    // readonly
    class A {
      readonly id = 'foo';
    }
    class A {
      _name = 'foo';
      get name() {
        return this._name;
      }
    }
    
    
    // 构造函数重载，不能声明返回值
    class Point {
      constructor(x:number, y:string);
      constructor(s:string);
      constructor(xs:number|string, y?:string) {
        // ...
      }
    }
    
    
    // 属性索引
    class MyClass {
      [s:string]: boolean |
        ((s:string) => boolean);
    
      get(s:string) {
        return this[s] as boolean;
      }
    }
    
    
    // 类的方法是一种特殊属性（属性值为函数的属性），所以属性索引的类型定义也涵盖了方法。
    // 如果一个对象同时定义了属性索引和方法，那么前者必须包含后者的类型。
    class MyClass {
      [s:string]: boolean;
      
      f() { // 报错
        return true;
      }
    }
    
    class MyClass {
      [s:string]: boolean | (() => boolean);
      
      f() { // OK
        return true;
      }
    }
    ```

  * 类的interface接口

    ```typescript
    interface Country {
      name:string;
      capital:string;
    }
    // 或者
    type Country = {
      name:string;
      capital:string;
    }
    class MyCountry implements Country {
      name = '';
      capital = '';
    }
    
    
    // interface 只是指定检查条件，如果不满足这些条件就会报错。它并不能代替 class 自身的类型声明。
    interface A {
      get(name:string): boolean;
    }
    
    class B implements A {
      get(s) { // s 的类型是 any
        return true;
      }
    }
    
    // implements关键字后面，不仅可以是接口，也可以是另一个类。这时，后面的类将被当作接口。
    class Car {
      id:number = 1;
      move():void {};
    }
    
    class MyCar implements Car {
      id = 2; // 不可省略
      move():void {};   // 不可省略
    }
    
    
    // 实现多个接口，尽量避免使用
    class Car implements MotorVehicle, Flyable, Swimmable {
      // ...
    }
    
    
    // TypeScript 不允许两个同名的类，但是如果一个类和一个接口同名，那么接口会被合并进类。
    class A {
      x:number = 1;
    }
    
    interface A {
      y:number;
    }
    
    let a = new A();
    a.y = 10;
    
    a.x // 1
    a.y // 10，如果在赋值之前读取，会返回undefined
    ```

  * 构造函数类型

    ```typescript
    class Point {
      x:number;
      y:number;
    
      constructor(x:number, y:number) {
        this.x = x;
        this.y = y;
      }
    }
    
    interface PointConstructor {
      new(x: number, y: number): Point;
    }
    
    function createPoint(
      PointClass:typeof Point,
      // or
      // PointClass: {new (x:number, y:number): Point},
      // or
      // PointClass: PointConstructor,
      x:number,
      y:number
    ): Point {
      return new PointClass(x, y);
    }
    
    ```

  * 结构类型原则：一个对象只要满足 Class 的实例结构，就跟该 Class 属于同一个类型。

    ```typescript
    class Foo {
      id!:number;
    }
    
    function fn(arg:Foo) {
      // ...
    }
    
    const bar = {
      id: 10,
      amount: 100,
    };
    
    fn(bar); // 正确
    
    
    // 如果两个类的实例结构相同，那么这两个类就是兼容的，可以用在对方的使用场合。
    class Person {
      name: string;
      age: number;  // 即使person有额外的属性
    }
    
    class Customer {
      name: string;
    }
    
    // 正确
    const cust:Customer = new Person();
    
    
    // 不仅是类，如果某个对象跟某个 class 的实例结构相同，TypeScript 也认为两者的类型相同。
    class Person {
      name: string;
    }
    
    const obj = { name: 'John' };
    const p:Person = obj; // 正确
    
    
    // 空类不包含任何成员，任何其他类都可以看作与空类结构相同。因此，凡是类型为空类的地方，所有类（包括对象）都可以使用。
    class Empty {}
    
    function fn(x:Empty) {
      // ...
    }
    
    fn({});
    fn(window);
    fn(fn);
    
    // NOT: 确定两个类的兼容关系时，只检查实例成员，不考虑静态成员和构造方法。
    
    ```

  * 继承其他类

    子类的同名方法不能与基类的类型定义相冲突，extends关键字后面不一定是类名，可以是一个表达式，只要它的类型是构造函数就可以了。

    ```typescript
    class A {
      protected x: string = '';
      protected y: string = '';
      protected z: string = '';
    }
    
    class B extends A {
      // 正确
      public x:string = '';
    
      // 正确
      protected y:string = '';
    
      // 报错
      private z: string = '';
    }
    
    
    // 继承一个表达式
    // 例一
    class MyArray extends Array<number> {}
    
    // 例二
    class MyError extends Error {}
    
    // 例三
    class A {
      greeting() {
        return 'Hello from A';
      }
    }
    class B {
      greeting() {
        return 'Hello from B';
      }
    }
    
    interface Greeter {
      greeting(): string;
    }
    
    interface GreeterConstructor {
      new (): Greeter;
    }
    
    function getGreeterBase(): GreeterConstructor {
      return Math.random() >= 0.5 ? A : B;
    }
    
    class Test extends getGreeterBase() {
      sayHello() {
        console.log(this.greeting());
      }
    }
    ```

    

  * 可访问性修饰符

    `public`（默认），`protected`，`private`，`#`

    ```typescript
    class A {
      private x = 1;
      // >= ES2022
      // #x = 1;
    }
    
    // 实例属性的简写
    class Point {
      constructor(
        public a: number,
        protected b: number,
        private c: number,
        readonly d: number
      ) {}
    }
    // 编译结果：
    class A {
        a;
        b;
        c;
        d;
        constructor(a, b, c, d) {
          this.a = a;
          this.b = b;
          this.c = c;
          this.d = d;
        }
    }
    
    ```

  * 静态成员

    ```typescript
    class MyClass {
      static x = 0;
      static printX() {
        console.log(MyClass.x);
      }
    }
    ```

  * 泛型类

    ```typescript
    class Box<Type> {
      contents: Type;
    
      constructor(value:Type) {
        this.contents = value;
      }
    }
    
    const b:Box<string> = new Box('hello!'); // <string>可以省略，因为可以自动推导
    ```

  * 抽象类和成员

    ```typescript
    abstract class A {
      abstract foo:string;
      bar:string = '';
      abstract execute():string;
    }
    ```

  * this问题

    TS允许函数增加一个名为`this`的参数，放在参数列表的第一位，用来描述内部`this`的类型。

    如果打开了这个noImplicitThis设置项，`this`的值推断为`any`类型，就会报错。

    ```typescript
    // 编译前
    function fn(
      this: SomeType,
      x: number
    ) {}
    // 编译后
    function fn(x) {}
    
    // this可声明为各种对象
    function foo(
      this: { name: string }
    ) {
      this.name = 'Jack';
      this.name = 0; // 报错
    }
    foo.call({ name: 123 }); // 报错
    ```

    `this`还可以作为返回值类型，表示当前类的实例对象

    ```typescript
    class Box {
      contents:string = '';
    
      set(value:string):this {
        this.contents = value;
        return this;
      }
    }
    ```

    有些方法返回一个布尔值，表示当前的`this`是否属于某种类型。这时，这些方法的返回值类型可以写成`this is Type`的形式，其中用到了`is`运算符。

    ```typescript
    class FileSystemObject {
      isFile(): this is FileRep {
        return this instanceof FileRep;
      }
    
      isDirectory(): this is Directory {
        return this instanceof Directory;
      }
    
      // ...
    }
    ```

    
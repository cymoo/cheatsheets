# TypeScript cheatsheet

## 基础

* `any` (top and bottom type), `unknown` (top type), `never` (bottom type)

* 基本类型：`boolean`, `string`, `number`, `bigint`, `symbol`, `object`, `undefined`, `null`
  
* 包装类型：`Boolean`, `String`, `Number`，不建议使用包装类型

  ```javascript
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

  ```javascript
  let age: number = 24;

  age = null;      // 正确
  age = undefined; // 正确
  ```

  * 但是如果打开编译选项`strictNullChecks`，它们就不能赋值给其他类型变量（除了`any`和`unknown`），打开此选项后，它们也不能相互赋值。

* 数组

  ```javascript
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

  ```javascript
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

  ```javascript
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

  ```javascript
  let x: 'hello'
  const x = 'https'
  ```

* 联合类型
  
  ```javascript
  let x: string | number
  let gender: 'male' | 'female'
  ```

* 交叉类型
  
  ```javascript
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

    ```javascript
    const a = { x: 0 };

    type T0 = typeof a;   // { x: number }
    type T1 = typeof a.x; // number
    ```

### TypeScript初步

#### TypeScript的安装

首先在项目目录下使用`npm init`生成`package.json`：

```shell
npm init -y
```

`TypeScript`可以通过`npm`进行安装，全局安装即可：

```shell
npm i typescript -g
```

安装完成后，我们通过使用命令`tsc --init`生成配置文件`tsconfig.json`：

```shell
tsc --init
```

#### 第一个TypeScript程序

接下来我们就可以编写`TypeScript`代码了，在项目目录新建一个目录`src`，在目录中新建一个`index.ts`文件，文件中编写第一行代码：

```typescript
const hello: string = 'hello TypeScript'
```

保存后我们需要对这个文件进行编译，因为`TypeScript`本身是不能运行的，需要编译成`JavaScript`文件：

```shell
tsc ./src/index.ts
```

编译完成后会发现生成了一个`index.js`文件，这就是编译生成的`JavaScript`代码，内容为：

```javascript
var hello = 'hello TypeScript';
```



接下来我们来配置构建工具`Webpack`，首先安装三个`npm`包：

```shell
npm i webpack webpack-cli webpack-dev-server -D
```

等待安装成功，我们在项目目录新建一个`build`目录，用来存放`Webpack`的配置文件,我们在里面新建四个配置文件，分别是`webpack.config.js（所有配置文件的入入口文件）`、`webpack.base.config.js（公共的配置）`、`webpack.dev.config.js（开发环境的配置）`、`webpack.pro.config.js（生产环境的配置）`，四个配置文件的具体代码如下：

```js
//webpack.base.config

const HtmlWebpackPlugin = require('html-webpack-plugin')
//这是一个Webpack插件，用于打包时帮助创建html文件

module.exports = {
    entry: './src/index.ts',	//配置项目入口文件
    output: {
        filename: 'app.js'	//配置项目出口文件
    },
    resolve: {
        extensions: ['.js', '.ts', '.tsx']	
/**webpack会按顺序解析对应的文件，如果多个文件共享相同的名称，但具有不同的扩展名，则webpack将解析该扩展名在数组中首先列出的文件，并跳过其余文件。*/
    },
    module: {
        rules: [{	//这里是配置loader，当遇到tsx结尾的文件就使用ts-loader解析
            test: /\.tsx?$/i,
            use: [{
                loader: 'ts-loader'
            }],
            exclude: /node_modules/	//排除node_modules下的文件
        }]
    },
    plugins: [	//这里使用插件，打包时帮助创建html文件
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ]
}
```

```js
//webpack.dev.config.js

module.exports = {
    devtool: 'cheap-module-eval-source-map'	//配置生产环境的source-map
}
```

```js
//webpack.pro.config.js

const {
    CleanWebpackPlugin
} = require('clean-webpack-plugin')	//导入了一个插件

module.exports = {
    plugins: [
        new CleanWebpackPlugin()	//使用这个插件帮助清理上次打包残留的文件
    ]
}
```

```js
//webpack.config.js

const merge = require('webpack-merge')	//引入了一个merge库，帮助我们合并配置文件
//将三个配置文件引入进来
const baseConfig = require('./webpack.base.config')
const devConfig = require('./webpack.dev.config')
const proConfig = require('./webpack.pro.config')

//根据环境变量来决定是什么环境的打包，并利用merge输出不同的合并配置
let config = process.NODE_ENV === 'development' ? devConfig : proConfig

module.exports = merge(baseConfig, config)
```

这几个配置文件我都写了详细的注释，如果对`Webpack`不太了解，可以查看我之前写的`Webpack笔记`。

既然上面用到了`ts-loader`，我们就需要安装一下`ts-loader`,并且在本地安装一下`TypeScript`：

```shell
npm i ts-loader typescript -D
```

接下来我们需要安装用到的`plugin`和`webpack-merge`：

```shell
npm i html-webpack-plugin clean-webpack-plugin webpack-merge -D
```

我们在`src`目录下生成一个`index.html`文件，用来当做打包生成的`html文件`的模板，具体如下，可以用编辑器快捷键快速生成：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>TypeScript-learn</title>
</head>
<body>
    <div class="app"></div>
</body>
</html>
```

接下来我们修改`npm script`：

```js
{
  "name": "TypeScript",
  "version": "1.0.0",
  "description": "",
  "main": "./src/index.ts",	//首先要修改入口文件
  "scripts": {
    "start": "webpack-dev-server --mode=development --config ./build/webpack.config.js", /**start命令使用dev-server,模式为开发模式，配置文件为webpack.config.js */
     "build": "webpack --mode=production --config ./build/webpack.config.js"
      //生产环境使用webpack打包，模式为生产环境，配置文件同上
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "clean-webpack-plugin": "^3.0.0",
    "html-webpack-plugin": "^3.2.0",
    "ts-loader": "^6.2.1",
    "typescript": "^3.6.4",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.9",
    "webpack-dev-server": "^3.9.0",
    "webpack-merge": "^4.2.2"
  }
```

接下来我们运行命令`npm start`，发现已经`webpack-dev-server`已经正常启动，打开`localhost:8080`发现页面已经可以正常显示。

我们修改一下`index.ts`：

```typescript
const hello: string = 'hello TypeScript'

document.querySelectorAll('.app')[0].innerHTML = hello
```

发现可以正常显示：

![hello-TypeScript](https://mexion.xyz/images/study-notes/TypeScript-learn/hello-TypeScript.jpg)

接下来测试下打包，使用`npm run build`命令，发现可以正常打包，在项目目录下生成了一个`dist`文件夹，文件夹中生成了`app.js`和`index.html`。

### TypeScript基础

#### TypeScript的类型

`TypeScript`是`JavaScript`的超集，所以它的类型大体是与`JavaScript`一致的。

在`ES6`以后，`JavaScript`有`6种基本类型`，分别是：

- `Number`
- `Boolean`
- `String`
- `Undefined`
- `Null`
- `Symbol`

其他的都是`引用类型`，比如：

- `Object`
- `Array`
- `RegExp`
- `Date`
- `Function`等

`TypeScript`在此基础之上增加了：

- `Void`
- `Any`
- `Never`
- `元组`
- `枚举`
- `高级类型`

#### 类型注解

所谓`TypeScript`,就是在原本`JavaScript`的基础之上拓展了类型系统，以对变量等进行类型约束，`TypeScript`使用`类型注解`来声明类型，它的作用相当于强类型语言中的类型声明，具体语法为`(变量/函数):type`。

我们在前面创建的`src`文件夹下新建一个`datatype.ts`，并在`index.ts`中引入它：

```typescript
//index.ts

import './datatype'
```



```typescript
//datatype.ts

// 原始类型 
//使用 (变量/函数):type 的方式声明
let bool: boolean = true
let num: number | undefined | null = 123
let str: string = 'abc'

//会报错，不能把一个Number赋值给一个String类型
 str = 123


// 数组
//定义数组有两种方式
方式一：
let arr1: number[] = [1, 2, 3]
方式二（这里用了泛型，官方已经定义好，泛型后面会学到）：
let arr2: Array<number> = [1, 2, 3]

//这里会报错，不能给一个只能添加Number的数组增加String
let arr2: Array<number> = [1, 2, 3, '4']
//改成下面这样既可（用到了联合类型，意思是两种类型皆可，两个类型之间用|分隔）：
let arr2: Array<number | string> = [1, 2, 3, '4']



// 元组（一种特殊的数组，它限定了数组的类型和个数）
//元组中存储的顺序是固定的，不能交换，长度也是固定的，不能改变
//这是一个元组，只能放两位，第一位固定为Number，第二位固定为String
let tuple: [number, string] = [0, '1']

//会报错，不能交换位置
let tuple: [number, string] = ['1', 0]

//元组的越界问题
tuple.push(2)	//虽然不能直接改变长度，但是可以push，也可以成功
console.log(tuple)	//输出为[0, '1'，2]
tuple[2]	/**会报错 Tuple type '[number, string]' of length '2' has no element at index '2'.*/



// 函数,需要为参数和返回值增加注解
//具体为 (param1: type, param1: type): returnType => do something...

let add = (x: number, y: number):number => x + y

//TypeScript有类型推断，所以返回值的number不写也是可以的
let add = (x: number, y: number) => x + y

//可以定义函数类型，这里的compute是一个函数类型，不是具体的函数实现
let compute: (x: number, y: number) => number
//函数实现
compute = (a, b) => a + b



// 对象，对象同样要增加注解
let obj: object = {x: 1, y: 2 }
//会报错，因为只是指定了obj的类型是Object，并未指定它具体有哪些属性
obj.x = 3

//改成这样既可：
let obj: { x: number, y: number } = { x: 1, y: 2 }
obj.x = 3



// symbol, symbol代表一个独一无二的值
let s1: symbol = Symbol()
//声明时不指定注解也是可以的，TS会类型推断
let s2 = Symbol()
console.log(s1 === s2)	//false symbol是独一无二的，不会相等



// undefined, null
//可以为一个变量定义为undefined和null，但是之后便不能再为它赋其他类型的值
let un: undefined = undefined
let nu: null = null
//报错
let un: undefined = 1

//undefined和null是任何类型的子类型，可以直接赋值给其他变量
//如果要使之不能赋值给其他类型，需要修改配置文件"strictNullChecks": true
//这时他们null和undefined只能赋值给void和它们各自，除非将变量设置为一种包含这两者的联合类型
num = undefined
num = null



// void
//在JS中，void是一种操作符，它会运行后面的表达式并返回undefined
//这里会返回undefined，用void 0 代替undefined本身的好处在于可以防止undefined本身被覆盖，因为undefined在JS中不是保留字，是可以被覆盖掉的
viod 0	
//在TS中，void类型表示没有没有任何类型，通常在函数中没有返回值就会返回void，void本身只能赋值为undefined和null
let noReturn = () => {}


// any，any表示任何类型，不指定类型时默认为any类型，可以为它赋任何值（除了never），这样就和JS没有什么区别了，不推荐使用
let x
x = 1
x = []
x = () => {}

// never，表示永远不会有返回值的类型，是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型
let error = () => {
    //抛出异常，不会有返回值
    throw new Error('error')
}
let endless = () => {
    //死循环，永远不会返回
    while(true) {}
}
```

#### 枚举类型

首先看一段代码：

```javascript
function initByRole(role) {
    if(role === 1 || role === 2) {
        // do sth
    } else if (role === 3 || role === 4) {
        // do sth
    } else if (role === 5) {
        // do sth
    }
}
```

假如这是一个系统，为不同的角色展现不同的界面和功能，这段代码存在的问题有：

- 可读性差： 很难记住数字的含义
- 可维护性差： 硬编码，数字所代表的角色可能被改变，成本和风险很大

如何解决这样的问题呢？使用`TypeScript`的枚举可以解决，什么是枚举？

**` 枚举是一组有名字的常量集合 `** 

就像手机中的通讯录，我们在打电话时会寻找联系人的名字而不会去寻找联系人的电话号码，我们不会去记住电话号码，只要去记住人名就可以了，电话号码可以变，但是人名是不会变的。

枚举有多种类型，我们在`src`下新建一个文件`enum.ts`，用于示范枚举：

```typescript

// 数字枚举
//这里有5个枚举成员，他们的取值从0开始，即Reporter为0，Developer为1，以此类推
//数字枚举可以用枚举成员的名字来索引，也可以用值来索引
enum Role {
    Reporter,
    Developer,
    Maintainer,
    Owner,
    Guest
}
console.log(Role.Reporter)	//输出0
console.log(Role)	
/** {0: "Reporter", 1: "Developer", 2: "Maintainer", 3: "Owner", 4: "Guest", Reporter: 0, Developer: 1, Maintainer: 2, Owner: 3, Guest: 4} */

//可以为枚举赋初始值，它会自己根据设置的初始值递增
enum Role {
    Reporter = 1,
    Developer,
    Maintainer,
    Owner,
    Guest
}
console.log(Role.Reporter)	//输出1



// 字符串枚举
enum Message {
    Success = '恭喜你，成功了',
    Fail = '抱歉，失败了'
}
console.log(Success) //输出 “恭喜你，成功了”
console.log(Message) //{Success: "恭喜你，成功了", Fail: "抱歉，失败了"} 这也意味着字符串枚举是不可以反向索引的



// 异构枚举 异构枚举即为数字枚举和字符串枚举混用(容易混淆，不推荐使用)
enum Answer {
    N,
    Y = 'Yes'
}
console.log(Answer) //{0: "N", N: 0, Y: "Yes"}

// 枚举成员
//会报错，枚举成员的值是只读类型，不能修改
Role.Reporter = 0

enum Char {
    // const member（常量成员）
    //分为没有初始值的情况、对已有枚举成员的引用、常量的表达式，在编译阶段就计算好
    a,
    b = Char.a,
    c = 1 + 3,
    // computed member（计算成员，非常量表达式，不会在编译阶段进行计算，保留到执行阶段）
    d = Math.random(),
    e = '123'.length,
    //computed member后面的枚举成员需要赋初始值，否则会报错
    f = 4
}

// 常量枚举
//为了避免在额外生成的代码上的开销和额外的非直接的对枚举成员的访问，我们可以使用 const枚举。 常量枚举通过在枚举上使用 const修饰符来定义。常量枚举在编译阶段会被删除。 常量枚举成员在使用的地方会被内联进来。
const enum Month {
    Jan,
    Feb,
    Mar,
    Apr = Month.Mar + 1,
    //去掉注释会报错，因为在编译阶段就要确定，所以常量枚举不允许包含计算成员
    // May = () => 5
}
let month = [Month.Jan, Month.Feb, Month.Mar]
//编译结果 var month = [0 /* Jan */, 1 /* Feb */, 2 /* Mar */]; 可以看到并没有上面定义的枚举，这是因为它在编译阶段被移除了。

// 枚举类型

//枚举和枚举成员都可以作为单独的类型存在
//所赋值要和枚举中的成员一致,比如E和F是数字枚举，那么就可以将Number赋值给属于它们类型的变量
enum E { a, b }
enum F { a = 0, b = 1 }
enum G { a = 'apple', b = 'banana' }

let e: E = 3
let f: F = 3
//尽管两个都是3，但是他们不能比较，比较会报错，除非他们都是相同的枚举类型，如两个都是E类型
// console.log(e === f)

let e1: E.a = 3
let e2: E.b = 3
let e3: E.a = 3
//会报错，不是同种类型
// console.log(e1 === e2)
//不会报错，是同种类型
// console.log(e1 === e3)

//字符串枚举只能赋值枚举成员的类型
//比如g1类型为G，那么可以将G.a和G.b赋值为g1
//g2由于是G.a类型，所以只能赋值G.a
let g1: G = G.a
let g2: G.a = G.a
```

#### 接口

`TypeScript`的核心原则之一是对值所具有的*结构*进行类型检查。 它有时被称做`“鸭式辨型法”`或`“结构性子类型化”`。 在`TypeScript`里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。总而言之，`TypeScript`在于为`JavaScript`拓展类型，那么既然上面说到的类型有类型注解约束类型，那么其他的内容又要怎么来约束呢？接口就是用来约束函数、对象、类的结构的。

我们首先创建一个文件`interface.ts`:

```typescript
//接口用interface关键字来定义，具体如下
//这里定义了一个接口list
interface List {
    readonly id: number;	//readonly表示只读属性，不能更改
    name: string;
    age?: number;	//属性后面跟问号表示可选属性，可有可无
}
//定义了一个Result接口，接口有一个data属性，它是一个只接受List的数组
interface Result {
    data: List[]
}
//render函数，只接收Result结构的参数
function render(result: Result) {
    result.data.forEach((value) => {
        console.log(value.id, value.name)
        if (value.age) {
            console.log(value.age)
        }
        //去掉注释会报错，只读属性不能修改
        // value.id++
    })
}
let result = {
    data: [
        //可以看到，即使我们在对象中新增了属性，但是不会报错，这就是“鸭式辨型法”
        //所谓鸭式辨型法，就是“不管它是什么东西，如果它走起来像鸭子，叫起来也像鸭子，那么它就是鸭子”
        //这在TS中也是类似的，它满足了List接口的所有条件，那么它就是List
        {id: 1, name: 'A', sex: 'male'},
        {id: 2, name: 'B', age: 10}
    ]
}
render(result)
//但是也有例外，假如render方法传入的不是一个变量，而是一个对象字面量，那么TS会对额外的属性进行检查
//下面这样会报错
render({
    data: [
        {id: 1, name: 'A', sex: 'male'},
        {id: 2, name: 'B', age: 10}
    ]
})
//绕过检查的方法有三种
//第一种是像像最上面一样使用变量保存再传递
//第二种是使用类型断言,如下，可以看到我们使用了as Result来表明我们知道这就是一个Result
render({
    data: [
        {id: 1, name: 'A', sex: 'male'},
        {id: 2, name: 'B', age: 10}
    ]
} as Result)
//类型断言的另一种方式，在传入对象之前增加了<Result>，这种方式和上面的等价
//但是推荐使用前一种，因为这种方法在React中会产生歧义
render(<Result>{
    data: [
        {id: 1, name: 'A', sex: 'male'},
        {id: 2, name: 'B', age: 10}
    ]
})
//第三种方式是使用索引签名，将List改成如下
//对应的意思是可以使用任意string去索引List会返回任意的内容，这样List就可以支持多个属性
//但是属性必须和定义的索引签名结构类型相一致
interface List {
    id: number;
    name: string;
    [x: string]: any;
}


//上面定义的接口都是固定的（除了最后修改的使用了索引签名）
//当不确定一个接口中有多少个属性的时候，就可以使用可索引类型的接口
//可索引类型的接口可以定义为使用数字索引或者字符串索引

//这是一个用数字索引的接口
//用数字可以索引字符串
interface StringArray {
    [index: number]: string
}
//比如一个字符串数组就完全符合这个接口
let chars: StringArray = ['a', 'b']

//接下来写一个用字符串索引的接口
interface Names {
    [x: string]: string;
}

//两种索引可以结合混用
//这样就可以使用字符串索引，也可以数字索引
//但是数字索引的返回值类型一定要是字符串索引返回值类型的子类型，因为JS为自动进行类型转换
//比如number是any的子类型
interface Names {
    [x: string]: any;
    [z: number]: number;
}



//定义函数接口
//可以像下面这样定义一个函数类型
let add: (x: number, y: number) => number
//可以用接口来定义如下
interface Add {
   (x: number, y: number): number
}
//两种定义方式是等价的

//一种更简洁的方式是使用类型别名，这里指的是为这个函数类型起的别名叫Add
//后面定义函数就可直接使用这个别名来作为注解
type Add = (x: number, y: number) => number
let add: Add = (a: number, b: number) => a + b

//类型别名的更多使用如下
type a = number
let num:a = 123

type name = "小明" | "小红" | "小强"
let xm:name = "小明"




//混合类型接口
//这个接口既定义了函数，也定义了属性和方法
interface Lib {
    (): void;	//表明这个lib是一个函数，不接收参数，也没有返回值
    version: string;	//它有一个version属性，是一个字符串
    doSomething(): void;	//有一个doSomething方法，没有返回值
}

//实现一些这个接口
let lib: Lib = (() => {}) as Lib
lib.version = '1.0.0'
lib.doSomething = () => {}

//封装一个函数以创建更多的实例
function getLib() {
    let lib = (() => {}) as Lib
    lib.version = '1.0.0'
    lib.doSomething = () => {}
    return lib;
}
let lib1 = getLib()
lib1()
let lib2 = getLib()
lib2.doSomething()
```



#### 函数

##### 函数定义

函数定义有四种方式：

```typescript
function add1(x: number, y: number) {
    return x + y
}

let add2: (x: number, y: number) => number

type add3 = (x: number, y: number) => number

interface add4 {
    (x: number, y: number): number
}
```

- 第一种使用了function关键字来进行定义，对参数进行了类型注解，返回值由于类型推断可以省略
- 第二种通过一个变量来定义一个函数类型
- 第三种使用了类型别名来定义一个函数类型
- 第四种使用了接口定义了一个函数接口

**需要注意的是后三种只是定义了函数的类型，但是没有具体的实现。**

在`JavaScript`中，对函数的参数个数是没有限制的，但在`TypeScript`中，**函数的形参和实参必须一一对应**，多一个或者少一个都是不行的，会直接报错。

##### 可选参数

有时，我们的需求是参数的可选的，而不是太过于死板，这时就可以使用到**`可选参数`**。

```typescript
function add5(x: number, y?: number) {
    return y ? x + y : x
}
add5(5)
```

可以看到在`参数y`的后面加了一个`?`表明这个参数是可选的。里面是一个`三元表达式`,如果有`参数y`就返回`x + y` ，否则返回`x`。

有一点需要注意的是，**`可选参数必须位于必选参数之后`**，如果可选参数位于必选参数之前，那么会直接报错。

##### 函数默认值

和`ES6`一致，**`函数的参数是可以有默认值的`**，具体如下：

```typescript
function add6(x: number, y = 0, z: number, q = 1) {
    return x + y + z + q
}
```

在调用时，如果在有默认值的参数后面还有参数，即使要使用参数的默认值，也不可以省略参数的传递，要为它传入`undefined`，这样形参与实参之间才能一一对应：

```typescript
add6(1, undefined, 3)
```

##### 剩余参数

上面的函数参数都是固定的，假如我们的需求是传入的参数不是固定的，需要怎么做呢？

与`ES6`一直，可以使用`剩余参数`，所谓剩余参数，其实就是使用了`拓展运算符...`：

```typescript
function add7(x: number, ...rest: number[]) {
    return x + rest.reduce((acc, cur) => acc + cur)
}
add7(1,2,3,4,5,6,7,8,9,10)	//55
```

可以看到我们将剩余的参数使用`拓展运算符`结合成了一个接收`Number`的数组，名称为`rest`（名称可以自己定），在函数内部，我们使用了数组的`reduce`方法对所有参数进行累加，再调用时我们传入了`1~10`，即`10`个参数，最终得到了正确的结果`55`。

##### 函数重载

`JavaScript`本身是个动态语言。 `JavaScript`里函数根据传入不同的参数而返回不同类型的数据是很常见的。那么在`TypeScript`中怎么做到这样呢？很多静态语言里都有`函数重载`的概念，含义是**两个函数名称相同，但是参数个数或者类型不同，这样就是函数重载。**

`TypeScript`中的函数重载这样表示：

```typescript
function add8(...rest: number[]): number;
function add8(...rest: string[]): string;
function add8(...rest: any[]) {
    let first = rest[0];
    if (typeof first === 'number') {
        return rest.reduce((acc, cur) => acc + cur);
    }
    if (typeof first === 'string') {
        return rest.join('');
    }
}
console.log(add8(1, 2))		//输出 3
console.log(add8('a', 'b', 'c'))	//输出 abc
```

为同一个函数提供多个函数类型定义来进行函数重载。 编译器会根据这个列表去处理函数的调用并进行正确的类型检查。

我们首先定义了两个同名的函数类型，但是他们的参数类型和返回值不同，一个接收`Number`类型的数组，返回值为`Number`类型，另一个定义为接收`String`类型的数组，返回值为`String`类型。

最后我们实现这个函数时会以同样的命名，并且将参数定义为接收`Any`类型的数组，在函数内部我们判断传入的参数是数字数组还是字符串数组，根据不同的参数来做不同的事情。在这个例子中，如果传入的是数字那么就返回累加值，如果是字符串就返回字符串的拼接。

`TypeScript`实现重载实际上是根据我们定义的**`重载列表`**，也就是我们上面定义的两个函数类型，会根据传参依次匹配每个定义，直到匹配成功使用匹配的函数定义，所以在定义重载的时候，一定要把最精确的定义放在最前面。注意最后的函数实现不是重载的一部分，重载列表只有上面定义的两个，一个接收`Number`类型的数组，一个接收`String`类型的数组。


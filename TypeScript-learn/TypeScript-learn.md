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
module.exports = (env, argv) => {
    let config = argv.mode === 'development' ? devConfig : proConfig
    return merge(baseConfig, config)
}
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

#### 类

在`ES6`之后，我们可以像传统的面向对象的语言一样使用`class`去创建一个类了。`TypeScript`中的类覆盖了`JavaScript`中的类，同时也引入了一些其他特性。

##### 类的实现

这是一个基本的`TypeScript`类：

```typescript
class Dog {
    constructor(name: string) {
        this.name = name
    }
    name: string
    run() {}
}
```

这里定义了一个`Dog`类，和`JavaScript`不同在于我们为属性和参数增加了类型注解，构造器`constructor`的返回值会自动推断为`Dog`，也就是这个类本身，方法`run`没有指定返回值，它的返回值默认为`void`。

主要注意的是，在类中定义的属性都是实例属性，在类中定义的方法都是原型方法：

```typescript
class Dog {
    constructor(name: string) {
        this.name = name
    }
    name: string
    run() { }
}

const dog = new Dog('ergou')

console.log(Dog.prototype)	// {run: ƒ, constructor: ƒ}
console.log(dog)	// Dog {name: "ergou"}
```

另外，在`TypeScript`中，实例的属性必须具有初始值，或者在构造函数中被初始化，或者将属性设置为可选属性：

```typescript
class Dog {
    constructor(name: string) {
        //如果像这样注释掉会报错，因为name没有初始值
        //this.name = name
    }
    name: string	//或者在这里直接赋初始值 || 或在这个属性后加？表示可选
    run() { }
}
```

##### 类的继承

我们在此基础上先定义一个`Dog`的子类`Shiba`：

```typescript
class Shiba extends Dog {
    constructor(name: String, color: string) {
        super(name)
        this.color = color
    }
    color: string
}
```

可以看到，我们使用`extends`关键字来继承父类，同时我们在构造函数中调用了`super函数`，`ES6` 要求，子类的构造函数必须调用一次 `super() 函数`。

注意：super 作为函数调用时，代表父类的构造函数。作为函数时，super() 只能用在子类的构造函数之中，用在其他地方就会报错。super 作为函数调用时，内部的 this 指的是子类实例；super 作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

然后我们可以给子类添加自己的属性，比如这里增加了一个`color`，同时必须赋初始值，我们定义了一个参数`color`，并在构造函数中为属性赋值。注意如果需要调用`this`，则一定要在`super`之后再调用。

##### 成员修饰符

成员修饰符是`TypeScript`对`JavaScript`的一个拓展。

修饰符有几种：

- `public`，即`公有成员`，类的所有属性默认都是`public`，意味着这些属性对所有人都是可见的
- `private`，即`私有成员`，它表明这个成员只能在这个类本身被调用，而不能被实例调用，也不能被子类调用
- `protected`，即`受保护成员`，它表明这个成员只能在类自身和子类被访问，而不能在类的实例中访问
- `readonly`，即`只读属性`，只能用在属性（非方法）之前，表明这个属性是只读的，只读属性一定要被初始化
- `static`，即`静态成员`，静态成员只能通过类名（包括子类）来调用，而不能通过实例调用

```typescript
class Dog {
    constructor(name: string) {
        this.name = name
    }
    name: string
    run() { }
    private pri() {}
    protected pro() {}
    readonly legs: number = 4
    static food: string = 'bones'
}

const dog = new Dog('ergou')
//报错，实例不能调用私有成员
dog.pril()
//报错，实例不能调用受保护成员
dog.pro()

class Shiba extends Dog {
    constructor(name: String, color: string) {
        super(name)
        this.color = color
        //报错，子类不能调用私有成员
        this.pri()
        //可以调用，子类可以调用受保护成员
        this.pro()
    }
    color: string
}
```



我们也可以给构造函数加上私有成员修饰符，这表明它不能被实例化，也不能被继承。

也可以给构造函数加上受保护成员修饰符，这表明这个类只能被继承而不能被实例化，相当于盛行了一个`基类`。



除了类的成员可以添加修饰符以外，构造函数的参数也可以添加修饰符，它的作用在于将参数自动变为实例的属性，这样就可以省略在类中的定义了，比如：

```typescript
class Shiba extends Dog {
    constructor(name: String, public color: string) {
        super(name)
        this.color = color
    }
    //可以注释掉了，参数已经加了修饰符，会自动将参数变为属性
    //color: string
}
```

##### 抽象类

`JavaScript`中没有`抽象类`的概念，`抽象类`是`TypeScript`对`ES`的拓展。

**抽象类，就是只能被继承，而不能被实例化的类。**

在之前的基础上，我们创建一个抽象类`Animal`： 

```typescript
abstract class Animal {
    eat() {
        console.log('eat')
    }
}

//报错，不能实例化
let animal = new Animal()

class Dog extends Animal {
    constructor(name: string) {
        super()
        this.name = name
    }
    name: string
    run() { }
}
let dog = new Dog('erha')
dog.eat()	//eat
```

我们用`Dog`类去继承这个抽象类，然后用`Dog`的实例去调用这个抽象类的方法，发现是可以成功调用的。

在抽象类中也可以不指定方法的具体实现，这就构成了一个抽象方法，具体实现如下：

```typescript
abstract class Animal {
    eat() {
        console.log('eat')
    }
    //定义了一个抽象方法，没有具体实现
    abstract sleep(): void
}

//报错，不能实例化
let animal = new Animal()

class Dog extends Animal {
    constructor(name: string) {
        super()
        this.name = name
    }
    name: string
    run() { }
    //在子类中具体实现
    sleep() {
        console.log('dog sleep')
    }
}
let dog = new Dog('erha')
dog.eat()	//eat
```

抽象类的好处就是可以抽离出一些事物的共性，有利于代码复用和扩展，另外，抽象类可以实现`多态`。所谓多态就是在父类中定义一个抽象方法，在每个子类中进行具体的不同实现。

我们可以具体实现以下多态，我们再定义一个类来继承抽象类`Animal`：

```typescript
class Cat extends Animal {
    sleep() {
        console.log('cat sleep')
    }
}

let cat = new Cat()
let animals: Animal[] = [dog, cat]
animals.forEach(animal => {
    animal.sleep()
})
//输出 dog sleep 和 cat sleep
```



##### this类型

类的成员方法可以直接返回一个`this`，这样就可以方便地实现`链式调用`。

```typescript
class Workflow {
    step1() {
        return this
    }
    step2() {
        return this
    }
}

new Workflow().step1().step2()
```

在继承时，`this类型`可以表现出多态（this是可变的，既可以是父类型也可以是子类型）：

```typescript
class Workflow {
    step1() {
        return this
    }
    step2() {
        return this
    }
}

class Myflow extends Workflow {
    next() {
        return this
    }
}

new Myflow().next().step1().next().step2()	//可以链式调用，返回的this为Myflow实例
```

这里的`this`其实与`JavaScript`表现一致。

##### 类和接口的关系

这是一个`类类型接口`，这个接口约束了一个类的属性和类型：

```typescript
interface Human {
    name: string
    eat(): void
}

class Asian implements Human {
    constructor(name: string) {
        this.name = name
    }
    name: string
    eat() {}
}
```

我们定义了一个类`Asian`去实现了这个接口，类实现接口要使用`implements`关键字，实现时必须要实现接口中所有的属性，类实现接口后增加自己的属性或方法是可以的。

接口只能约束类的共有成员，另外，接口也不能约束类的构造函数。

##### 接口的继承

接口可以像类一样相互继承，并且一个接口可以继承多个接口：

```typescript
interface Man extends Human {
    run(): void
}

interface Child {
    cry(): void
}
    
interface Boy extends Man, Child {}
    
let boy: Boy = {
    name: 'lala',
    run() {},
    eat() {},
    cry() {}
}
```

继承多个接口需要使用逗号`,`分隔。

接口除了可以继承接口之外，还可以继承类，相当于接口把类的成员都抽象出来：

```typescript
class Auto {
    state = 1
}

interface AutoInterface extends Auto {}

class C implements AutoInterface {
    state = 1
}

//Auto子类继承接口
class Bus extends Auto implements AutoInterface {}
```

#### 泛型

很多时候，我们希望一个类或者一个函数能够支持多种数据类型，而不是固定的类型，这该怎么实现呢？`泛型`就是为了解决这个问题而出现的。

##### 泛型函数

假如没有泛型：

```typescript
//这是一个打印函数
function log(value: string): string {
    console.log(value)
    return value
}
//这个函数接收一个字符串，打印这个字符串并返回这个字符串
```

如果我们修改了需求，在此基础之上我们希望可以它可以接受一个字符串数组，这时可以通过以下几种方式来实现：

```typescript
function log(value: string): string
function log(value: string[]): string[]
function log(value: any) {
    console.log(value)
    return value
}
//或者使用联合类型
function log(value: string | string[]): string | string[] {
    console.log(value)
    return value
}

```

如果再次更改需求，希望这个函数可以接收任意类型的参数：

```typescript
//直接使用any类型
function log(value: any) {
    console.log(value)
    return value
}
```

但是问题来了，这样修改之后我们丢失了对类型的约束，使用any相当于我们直接使用了`JavaScript`，同时，这样也使函数参数的类型和返回值的类型无法保持一致。

现在，使用`泛型`就可以解决这样的问题，什么是泛型？

> 泛型： 不预先确定数据类型，具体是什么类型在使用时确定。

我们现在改造一下上面写的`log`函数：

```typescript
function log<T>(value: T): T {
    console.log(value)
    return value
}
```

可以看到我们在圆括号前新增了一对尖括号，并在里面定义了一个`T`，这个`T`就是泛型，我们将参数的类型和返回值全部定义为`T`，这个`T`只是一个代表名称，具体是什么类型由我们在使用时指定，这样就可以做到类型一致，并且可以传递任何类型。

如何调用这个函数呢？

```typescript
//第一种，调用时直接指定类型，比如这里指定类型为string数组
log<string[]>(["a","b"])
//第二种，使用类型推断，直接调用，不指定类型(推荐)
log(["a", "b"])
```

##### 泛型函数类型

不仅可以用泛型来定义一个函数，也可以定义一个函数类型：

```typescript
type Log = <T>(value: T) => T

let myLog: Log = log
```

##### 泛型接口

泛型同样可以用在接口中:

```typescript
interface Log {
    <T>(value: T): T
}
```

这个定义的泛型接口和上面类型别名定义的函数的方式等价。

泛型不仅可以约束函数，也可以约束接口的其他成员，方法为将泛型放在接口名称后面：

```typescript
//这样接口的所有属性都会受到泛型的约束
interface Log<T> {
    (value: T): T
}
//在实现时也必须指定类型
//这样myLog只能接受数字
let myLog: Log<number> = log
```

如果不指定类型，我们可以指定接口的定义中，为泛型指定一个默认类型：

```typescript
interface Log<T = string> {
    (value: T): T
}

let myLog: Log = log
myLog("1")
```

##### 泛型类

泛型也可以约束类的成员：

```typescript
//把泛型放在类名称之后，就可以约束所有成员
class Log<T> {
    run(value: T) {
        console.log(value)
        return value
    }
}
```

需要注意的是，`泛型`不能应用于类的静态成员：

```typescript
class Log<T> {
    //会报错，静态成员不能引用类型参数
    static run(value: T) {
        console.log(value)
        return value
    }
}
```

实例化时我们可以显式传入`T`的类型：

```typescript
let log1 = new Log<number>()
log1.run(123)	//只能传入number
```

如果不显式指定类型，那么可以传入任意类型的值：

```typescript
let log2 = new Log()
log2.run("1")
log2.run(1)
```

##### 泛型约束

我们在打印时，不仅希望打印参数，也希望可以打印参数的某个值，比如例子中的`length`：

```typescript
function log<T>(value: T): T {
    //会报错，T类型上不存在length属性
    console.log(value, value.length)
    return value
}
```

这时候就要用到泛型约束这个概念。

我们要先预定义一个接口，这个接口有一个`length`属性，然后我们让`T`去继承这个接口：

```typescript
interface Length {
    length: number
}

function log<T extends Length>(value: T): T {
    console.log(value, value.length)
    return value
}
//数组和字符串都有length属性，传入指定对象也可以
log([1,2,3])
log(“你好”)
log({length: 2})
```

这样就可以通过类型检查，`T`继承了`Length`接口，就表示`T`受到了约束，不再是任何类型都可以传了，而是必须拥有`length`属性。

#### 类型检查机制

> 类型检查机制： TypeScript 编译器在做类型检查时，所秉承的一些原则，以及表现出的一些行为，以辅助开发、提高开发效率。

`TypeScript`的类型检查机制体现在：

- 类型推断
- 类型兼容
- 类型保护



##### 类型推断

**类型推断使我们可以不指定变量的类型，`TypeScript`会根据某些规则自动推断类型。**

```typescript
//基础类型推断

//定义变量时不指定类型，会自动推断为any类型
let a	//any类型
//赋了初始值，那么自动推断为这个值的类型
let a = 1	//number类型
let a = []	//类型为 any[]
let a = [1]	//类型为 number[]
//在为函数设置默认参数时也会自动推断
let func = (x = 1) => x + 1		//会推断X和返回值都为number类型


//最佳通用类型推断

//当需要从多个类型中推断出类型的时候，TS会尽可能推断一个兼容所有类型的通用类型
let b = [1, null]	//由于null和1互不兼容，所以会推断为 (number | null)[]

//上下文推断

//从左到右进行推断，比如事件处理
window.onkeydown = (e) => {}	//这会根据事件推断e为KeyboardEvent类型
```



**有时候可能`TS`的类型推断不符合你的预期，并且你认为你对变量的类型完全了解，那么你可以通过`类型断言`来断言变量的类型。**

```typescript
let foo = {}
//会报错，提示foo的类型{}上没有bar这个属性
foo.bar = 1

//通过类型断言来断言这个类型
//先定义一个接口
interface Foo {
    bar: number
}
//进行断言，断言为Foo类型
let foo = {} as Foo
foo.bar = 1
```

但是类型断言不能滥用，不然可能出现意想不到的错误。

所以推荐在定义时就指定类型，而不是使用类型断言来断言：

```typescript
interface Foo {
    bar: number
}

let foo: Foo = {}
foo.bar = 1
```



##### 类型兼容

什么是类型兼容？

**当一个类型Y可以被赋值给另一个类型X时，我们就可以说类型X兼容类型Y。**

```typescript
interface X {
    a: any
    b: any
}

interface Y {
    a:any
    b: any
    c: any
}

let x: X = {a: 1, b: 2}
let y: Y = {a: 1, b: 2, c: 3}
//可以赋值，因为x兼容y
x = y
//报错，y不兼容x
y = x
```

可以看到，只要一个接口的属性完全满足另一个接口的属性（即使另一个接口有额外属性），那么这个接口就兼容另一个接口，在这个例子中`X`完全满足`Y`，所以`X`兼容`Y`，这也体现了“鸭式辨型法”这个原则。

接下来是函数兼容性，函数兼容性一般发生在函数赋值或者函数作为参数传递的情况下：

```typescript
type Handler = {a: number, b: number} => void
function hof(handler: Handler) {
    return handler
}
//要做到函数类型兼容有以下几种情况
//1) 目标函数类型要多于兼容函数类型的参数个数（多的不能传给少的）
let handler1(a: number) => {}
//可以传入hof，因为handle1参数个数比Handler类型少，所以兼容Handler
hof(handler1)

let handler2(a: number, b: number, c: number) => {}
//不能传入hof，因为handle2参数比Handler类型多，不兼容Handler
hof(handler2)

//其次是可选参数和剩余参数的情况(多的不能传给少的)
let a =(p1: number, p2: number) => {}
let b = {p1?: number, p2?: number} => {}
let c = (...args: number[]) => {}
//可以赋值，固定参数兼容可选参数和剩余参数
a = b
a = c
//报错，可选参数不兼容剩余参数
b = c
//报错，可选参数不兼容固定参数
b = a
//可以赋值，剩余参数可以兼容可选参数和固定参数
c = a
c = b

//2)参数类型必须要匹配才能兼容
let handler3 = (a: string) => {}
//会报错，类型不匹配
hof(handler3)

interface Point3D {
    x: number,
    y: number,
    z: number
}

interface Point2D {
    x: number
    y: number
}

let p3d = (point: Point3D) => {}
let p2d = (point: Point2D) => {}
//可以赋值，参数少的可以赋值给参数多的
p3d = p2d
//报错，参数多的不能赋值给参数少的
p2d = p3d

//3)返回值类型（成员少的兼容成员多的）（可以参照鸭式辨型法）
let f = () => ({name: 'Alice'})
let g = () => ({name: 'Alice', location: 'Beijing'})
//可以赋值，f的返回值是g返回值的子类型
f = g
//报错
g = f

//函数重载
function overload(a: number, b: number): number
function overload(a: string, b: string): string
function overload(a: any, b: any): any {}
//实现函数要比重载列表的函数的参数少，并且参数和返回值类型要兼容重载列表的参数和返回值类型

```

然后是的枚举兼容性：

```typescript
//首先数字枚举和数值类型是兼容的
enum Fruit{ Apple, Banana }
enum Color{ Red, Yellow }
//可以赋值，数字枚举和数值完全兼容
let fruit: Fruit.Apple = 3
let num: number = Fruit.Apple

//但是枚举之间是完全不兼容的
//报错，枚举之间完全不兼容
let color: Color.Red = Fruit.Apple
```

类兼容性：

```typescript
//类之间兼容是只比较结构
//需要注意的是： 静态成员和构造函数是不参与比较的
class A {
    constructor(p: number, q: number) {}
    id: number = 1
}

class B {
    static s = 1
    constructor(p: number) {}
    id: number = 2
}

let aa = new A(1,2)
let bb = new B(1)

//两个实例完全兼容，因为不比较构造函数和静态成员，他们都有共同的实例属性
aa == bb
bb == aa

//如果一个类中有私有成员（private），那么他们是不兼容的，但是它子类的实例和父类实例兼容
class A {
    constructor(p: number, q: number) {}
    id: number = 1
    private name: string = 'aaa'
}

class B {
    static s = 1
    constructor(p: number) {}
    id: number = 2
    private name: string = 'bbb'
}

let aa = new A(1,2)
let bb = new B(1)
//报错，私有成员不兼容
aa = bb
bb == aa

class C extends A {}
let cc = new C(1,2)
//可以相互兼容
aa = cc
cc = aa
```

最后是泛型兼容性：

```typescript
//当泛型接口没有成员时两者兼容
interface Empty<T> {
    
}
let obj1: Empty<number> = {}
let obj2: Empty<string> = {}
obj1 = obj2
//有成员时不兼容
interface Empty<T> {
    
}
let obj1: Empty<number> = {}
let obj2: Empty<string> = {}
//报错
obj1 = obj2


//泛型函数
//这里定义了两个完全相同的泛型函数
let  log1 = <T>(x: T): T => {
    console.log('x')
    return x
}
let log2 = <U>(y: U): U => {
    console.log('y')
    return y
}
//可以赋值，两者兼容
log1 = log2
```



**最后的总结：**

- **结构之间兼容： 成员少的兼容成员多的（属性多的可以赋值给属性少的）**

- **函数之间兼容： 参数多的兼容参数少的（参数少的可以赋值给参数多的）**



##### 类型保护

> 类型保护： `TypeScript`能够在特定的区块中保证变量属于某种确定的类型。可以在此区块中放心引用此类型的属性或者调用此类型的方法。

这是一点`TS`代码，这里定义了一个枚举和两个类一个函数，运行这个函数时如果判断到类型是强类型就生成一个`Java实例`，否则生成一个`JavaScript实例`，并且运行他们的打印方法，假如没有类型保护：

```typescript
enum Type { Strong, Week }

class Java {
    helloJava() {
        console.log('Hello Java')
    }
}

class JavaScript {
    helloJavaScript() {
        console.log('Hello JavaScript')
    }
}

function getLanguage(type: Type) {
    let lang = type === Type.Strong ? new Java() : new JavaScript()
    //报错，提示Java | JavaScript类型上没有helloJava这个方法
    if(lang.helloJava) {
        //报错
        lang.helloJava()
    } else {
        //报错
        lang.helloJavaScript()
    }
    return lang
}

```

因为这时`TypeScript`会判断`lang`的类型为`Java`和`JavaScript`的联合类型，但是因为我们传入的类型不确定，所以它不能判断具体是哪一种类型，如果我们使用`as`为每个报错的地方都加上类型断言，虽然可以解决问题，但是过于繁琐，并且代码变得臃肿和难以阅读。

而类型保护机制就是为了解决这个问题，它可以对类型提前做出预判，生成类型保护区块具体有以下几种方法：

```typescript
//1)使用 instanceof 关键字 (可以判断某个实例是不是属于某个类)
//改造代码如下：
function getLanguage(type: Type) {
    let lang = type === Type.Strong ? new Java() : new JavaScript()
    if(lang instanceof Java) {
        //这个区块就是类型保护区块，TS可以确定在这个区块中lang一定为Java类
        lang.helloJava()
    } else {
        //在这个区块中一定是JavaScript的实例
        lang.helloJavaScript()
    }
    return lang
}

getLanguage(Type.Strong)

//2)使用 in 关键字 （可以判断某个属性是否存在于某个对象）
//改造代码如下：
class Java {
    helloJava() {
        console.log('Hello Java')
    }
    //在Java类中添加一个属性
    java: any
}

class JavaScript {
    helloJavaScript() {
        console.log('Hello JavaScript')
    }
    //在JS类中添加一个属性
    javascript: any
}

function getLanguage(type: Type) {
    let lang = type === Type.Strong ? new Java() : new JavaScript()
    if('java' in lang) {
        //这个区块就是类型保护区块，TS可以确定在这个区块中lang一定为Java类的实例
        lang.helloJava()
    } else {
        //在这个区块中一定是JavaScript的实例
        lang.helloJavaScript()
    }
    return lang
}

//3) 使用 typeof 关键字 (可以判断基本类型)
//代码如下：
(function (p: string | number) {
    if(typeof x === 'string') {
        //这里面一定是string，那就一定会有length属性
        x.length
    } else {
        //一定是number，一定会有toFixed方法
        x.toFixed(2)
    }
})("hello")
```

同时我们也可以创建一个类型保护函数来创建类型保护：

```typescript
//先定义一个类型保护函数
function isJava(lang: Java | JavaScript): lang is Java {
    return (lang as Java).helloJava !== undefined
}
//可以看到函数可以传入一个联合类型参数，但是返回值为一个特殊的类型，这种类型我们通常称为`类型谓词`，语法为 （ 参数 is 类型 ）
//然后我们在函数体中进行判断

function getLanguage(type: Type) {
    let lang = type === Type.Strong ? new Java() : new JavaScript()
    if(isJava(lang)) {
        //这个区块就是类型保护区块，TS可以确定在这个区块中lang一定为Java类的实例
        lang.helloJava()
    } else {
        //在这个区块中一定是JavaScript的实例
        lang.helloJavaScript()
    }
    return lang
}
```

#### 高级类型

##### 交叉类型

**将多个类型合并为一个类型，这个合并的类型将具有所合并类型的所有特性。**

```typescript
interface DogInterface {
    run(): void
}
interface CatInterface {
    jump(): void
}
//使用 & 符号以生成交叉类型，这个类型将具有两个类型的所有属性。
let pet: DogInterface & CatInterface = {
    run() {},
    jump() {}
}

```

##### 联合类型

**当类型并不确定，可以赋为多个类型中的任意一个，可以使用联合类型。**

```typescript
//string | number是一个联合类型
//那么这个类型就可以赋值为string或者number
type StringOrNum = string | number
//可以看到初始赋值为number，改变为string也是可以的
let a: StringOrNum = 1
a = "hello"

//字面量联合类型
//1)字符串
//有时我们不仅需要限定变量的类型，还要限定它的取值在一定范围内
type StringType = '小红' | '小明'
//可以赋值
let nickName:StringType = '小红'
nickName = '小明'
//报错，‘芳’在 '小红'和'小明'两者范围内
nickName = '小芳'

//2)数字
type NumType = 1 | 2
//可以赋值
let ANum: NumType = 1
ANum = 2
//报错，3不在两者范围内
ANum = 3
```

##### 索引类型

首先看没有索引类型的情况：

```typescript
let obj = {
    a: 1,
    b: 2,
    c: 3
}

function getValues(obj: any, keys: string[]){
    return keys.map(key => obj[key])
}
console.log(getValues(obj, ['a', 'b']))	// 输出 [1,2]
console.log(getValues(obj, ['e', 'f']))	//输出 [undefined, undefined]
```

可以看到我们访问对象中没有的属性时，输出了两个`undefined`，而且并没有报错，这是我们不想遇到的情况，这时就可以使用`索引类型`来解决。

首先要说两个概念：

- `keyof`：索引类型查询操作符（index type query operator）
- `T[K]`：索引访问操作符（indexed access operator）



`keyof T`取类型`T`上的所有`public`**属性名**构成联合类型，例如：

```typescript
interface Obj {
    a: number,
    b: string
}

let key: keyof Obj	//类型 "a" | "b"
```

`T[K]`取`T`类型上`K`**属性**的类型：

```typescript
interface Obj {
    a: number,
    b: string
}

let value: Obj['a']	//类型 number
```

现在来改造下前面的例子：

```typescript
let obj = {
    a: 1,
    b: 2,
    c: 3
}

function getValues<T, K extends keyof T >(obj: T, keys: K[]): T[K][]{
    return keys.map(key => obj[key])
}
console.log(getValues(obj, ['a', 'b']))	// 输出 [1,2]
//报错，不能将类型“string”分配给类型“"a" | "b" | "c"”
console.log(getValues(obj, ['e', 'f']))	
```

可以看到，我们使用泛型约束了`obj`和`keys`，泛型`K`的类型取决于`T`类型的索引类型，`keyof`关键字会将`T`类型上的所有`key`构成联合类型，函数的返回值是一个由`T[K]`类型组成的数组。

然后我们将`obj`传入，`T`的类型即为`obj`的类型`{a: number, b: number, c: number}`，`K`的类型则为`T`的`key`组成的联合类型`'a' | 'b' | 'c'`，这也意味着`keys`的成员类型只能为`obj`的`key`，`e`、`f`不是`obj`的`key`，故报错。

##### 映射类型

> 映射类型是一种从旧类型中创建新类型的方式。 

假如要把一个接口的属性全部变为只读，可以这样实现：

```typescript
interface Obj {
    a: string
    b: number
    c: boolean
}

//这时ReadonlyObj的类型将为以Obj为基础的，将属性全改为只读的类型
//Readonly为TS内置的泛型接口
type ReadonlyObj = Readonly<Obj>
```

这个泛型接口的定义方式如下：

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P]
};
```

可以看到这个泛型接口接收一个`T`，然后在其中进行了对原成员的修改，在之前增加了`readonly`，

`P in keyof T`这段实际上进行了一次`for ... in`操作，`keyof T`将获取到所有属性的联合类型，索引签名的返回值为`T[P]`，即使用索引访问操作符获取了`T`上`P`所指定的类型。

还有把一个接口的属性变为可选的，可以这样操作：

```typescript
interface Obj {
    a: string
    b: number
    c: boolean
}
type PartialObj = Partial<Obj>
```

实现方式如下：

```typescript
//TS 已经内置
type Partial<T> = {
    [P in keyof T]?: T[P]
}
```

如果要将一个接口抽取出一个子集，可以这样操作：

```typescript
interface Obj {
    a: string
    b: number
    c: boolean
}

type PickObj = Pick<Obj, 'a' | 'b'>

//实现方式
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

这时，`PickObj`将为`Pick`抽取出来的一个子集，只会包含`a`和`b`两个成员。

还可以这样使用映射类型：

```typescript
interface Obj {
    a: string
    b: number
    c: boolean
}

/*type RecordObj = {
    x: Obj
    y: Obj
}*/
type RecordObj = Record<'x' | 'y', Obj>

//实现方式
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

这样`RecordObj`的成员由第一个参数所指定，而他们的类型有第二个参数所指定，这里为`Obj`。

##### 条件类型

> 条件类型是一种由条件表达式所决定的类型。

他的形态为`T extends U ? X : Y`，这里的意思是假如`T extends U（T可以赋值给U）`那么类型为`X`，否则为`Y`，其实就是一个三元表达式。

下面是一个例子： 

```typescript
//这里使用了三元表达式的多重判断，三元表打死会依次判断以匹配返回不同的字符串
type TypeName<T> =
	T extends string ? "string":
    T extends number ? "number":
    T extends boolean ? "boolean":
    T extends undefined ? "undefined":
    T extends Function ? "function":
    "object"

type T1 = TypeName<string>	// "string"
type T2 = TypeName<string[]>	// "object"
type T3 = TypeName<undefined>	// "undefined"
type T4 = TypeName<null>	// "object"

//分布式条件类型
形态为(A | B | C) extends U ? X : Y
//等价于
(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)

type T5 = TypeName<string | string[]>	// "string" | "object"
```

利用这个特性，我们来写一个过滤类型：

```typescript
type Diff<T, U> = T extends U ? never : T
type T6 = Diff<'a' | 'b' | 'c', 'a' | 'e'>	// "b" | "c"
```

这个类型可以将`T`中可以赋值给`U`的类型给过滤掉，我们来拆解一下：

```typescript
//T6可以等价为
Diff<'a', 'a' | 'e'> | Diff<'b', 'a' | 'e'> | <'c', 'a' | 'e'>
//结果为
 //never | "b" | "c"
//never会被忽略掉，最后结果为
//"b" | "c"

```

我们也可以利用`Diff`将`null`和`undefined`给过滤掉：

```typescript
type NotNull<T> = Diff<T, undefined | null>
type T7 = NotNull<string | number | undefined | null>	// string | number
```

其实这两个类型`TypeScript`已经内置定义好了，`Diff`在官方定义中为`Exclude<T, U>`， `NotNull`在官方定义中为`NomNullable<T>`，平时用到直接调用官方的即可。

官方还定义了一些其他的条件类型，比如：

- `Extract<T, U>`，这与`Exclude`相反，它可以将`T`可以赋值给`U`的类型抽取出来。
- `ReturnType<T>`，可以获取一个函数返回值的类型。

### TypeScript工程

#### 模块系统

`TypeScript`对`ES6`和`CommonJS`两种模块系统都有很好的支持，我们基本可以沿用以前的写法。但两者不要混用，如果出现混用两个模块之间会出现不兼容的情况。

- 导出：`ES6`允许同时存在`export default`和`export`多个变量，而`CommonJS`只允许有一种形式的导出，其中一种会把另外一种覆盖掉。
- 导入：`ES6`可以按需导入也可以全部导入，而`CommonJS`只能全部导入。

如果在`ES6`模块中抛出数据，在非`ES6`模块中导入，就会出现问题。因此尽量不要混用不同的模块化系统。如果迫不得已，可以使用TS提供的兼容性语法：

```
`// 导出``export` `= a;``// 导入``import` `c4 = require(``'../es6/c'``);``/*``1.如果使用以上方法导出，此文件不允许有其它形式的导出``2.以上形式的导出的数据，不仅可以用以上语法导入，还可以用es6的方式导入。前提是tsconfig.json中的"esModuleInterop"：true配置项要开启。``*/`
```

#### 命名空间

在`JavaScript`中，命名空间可以有效地避免全局污染，但在`ES6`引入了模块系统之后，命名空间就很少被提及了，但是`TypeScript`依然实现了这个特性。如果使用了全局的类库，命名空间仍是一个好的解决方案。

```typescript
//使用namespace关键字定义了一个命名空间
namespace Shape {
    const p1 = Math.PI
    //在里面用export导出需要的类、接口、变量、类型或者方法
    export function cricle(r: number) {
        return pi * r ** 2
    }
}
```

两个文件中同一名称的命名空间是共享的，命名空间的调用方法为：

```typescript
Shape.cricle(1)
```

可以看到这样每次使用都要加上命名空间的前缀，这时我们可以给一个别名：

```typescript
import cricle = Shape.cricle

cricle(2)
```

**命名空间语法中的export和import和模块中的是不同的。**



在一个文件中导入另一个文件中的命名空间，需要使用三斜杠：

```typescript
/// <reference path="a.ts"/>
```

需要注意的是**命名空间不要随意使用，不要在一个模块中使用命名空间，而最好在一个全局的环境中使用。**

#### 声明合并

>  `TypeScript`会把程序的多个地方具有相同名称的声明合并成一个，这样可以将散落在各处的重名声明合并在一起。

##### 接口声明合并

```typescript
interface A {
    x: number
}
interface A {
    y: number
}
//声明合并，a必须具有两个分散接口的所有成员
let a: A = {
    x: 1,
    y: 2
}

//如果两个分散接口中拥有同名属性，如果他们类型相同则不会报错，反之报错
interface A {
    x: number
    y: string
}
interface A {
    y: number
}
//报错，类型冲突
let a: A = {
    x: 1,
    y: 2
}

//如果是函数成员发生重名，则会使用函数重载
interface A {
    x: number
    foo(bar: number): number	//3
}
interface A {
    y: number
    foo(bar: string): string	//1
    foo(bar: number[]): number[]	//2
}

let a: A = {
    x: 1,
    y: 2,
    foo(bar: any) {
        return bar
    }
}

//那么函数重载列表的顺序是怎样的呢？
//1)接口内部按定义顺序排列
//2)接口之间后面的接口会排在前面
//最后排序如上面的注释

//但是也有例外
//假如函数的参数是一个字符串字面量，这个函数声明就会提升到列表的最顶端
//下面是一个例子，顺序如注释
interface A {
    x: number
    foo(bar: number): number	//5
    foo(bar: 'a'): number	//2
}
interface A {
    y: number
    foo(bar: string): string	//3
    foo(bar: number[]): number[]	//4
	foo(bar: 'b'): number	//1
}

```

##### 命名空间和函数合并

```typescript
function Lib() {
}
namespace Lib{
   export let version = '1.0'
}
console.log(Lib.version); // 相当于为函数Lib添加了属性
```

##### 命名空间和类合并

```typescript
class C{
}
namespace C{
   export let state = 1
}
console.log(C.state); // 相当于为类C添加了state属性
```

此外，命名空间也可以和枚举合并，同样的也是为枚举增加属性或者方法。

**注意：**在命名空间与类、函数进行生命合并的时候，一定要将命名空间放在类、函数之后。否则报错。

#### 声明文件

假如我们要在`TypeScript`中引入`jQuery`，我们需要这样做：

首先安装`jQuery`：

```typescript
npm i jquery
```

`jQuery`是一种`UMD`库，可以通过全局的方式引入也可以通过模块化的方式引入，我们使用模块化的方式引入：

```typescript
import $ from 'jquery'
```

这里就会直接报错，提示无法找到模块`jquery`的声明文件。

这是因为`jQuery`本身是用`JavaScript`编写的，我们在使用非`TypeScript`编写的类库时必须为这个类库编写一个声明文件，对外暴露它的`API`，有时候这些类库的声明文件包含在源码中，但有时候需要额外的安装，比如`jQuery`。

大多数类库的声明文件其实社区都已经帮助我们编写好了，只需要在使用类库时安装指定类库的声明文件包而已。这种包的命名一般为`@types/类库名`，以`jQuery`为例：

```typescript
npm i @types/jquery -D
```

安装完成后就可以消除错误提示，可以在`TypeScript`中使用`jQuery`了。



##### 编写声明文件

我们在`TypeScript`中使用`JavaScript`类库时，首先要考虑的是社区是否已经有这个声明文件，我们可以通过这个[网站](microsoft.github.io/TypeSearch/ )进行查询。假如社区没有特定类库的声明文件，则需要自己去写一个。

> 首先看全局库:

我们可以先新建一个文件`global-lib.js`，代码如下：

```javascr
//一段普通的JS代码，定义了函数和属性
function globalLib(option) {
console.log(options)
}

globalLib.version = '1.0'

globalLib.doSomething = function() {
	console.log('globalLib do something')
}
```

因为这是一个全局库，然后我们可以直接在`html`文件中使用`script`标签引入这个文件，我们在`index.ts`中直接调用这个类库中的方法，代码如下：

```typescript
//报错，找不到名称globalLib
globalLib({x: 1})
```

这里报错的原因就是我们没有为这个类库编写声明文件，声明文件一般用`类库名.d.ts`命名，现在我们编写声明文件`global-lib.d.ts`，代码如下：

```typescript
//我们使用了declare关键字，它可以为一个外部变量提供类型声明

//我们在类库中定义了一个全局的函数，我们这里也定义了一个对应的声明
//声明它是一个函数，名称为globalLib，接收一个参数，参数类型为global.options，没有返回值
declare function globalLib(options: globalLib.Options): void

//这里我们用了命名空间，定义了同名的命名空间
//函数和命名空间会合并，将命名空间中的属性和方法挂载到同名函数之上
declare namespace globalLib {
    //命名空间中有一个属性
    const version: string
    //一个方法
    function doSomething(): void
    //还有一个接口，这个接口名为Options，成员是一个索引签名，key为string，返回any
    intereface Options {
        [key: string]: any
    }
}
```

声明文件编写完毕，可以看到`index.ts`中的报错已经消除，我们可以直接使用这个类库的方法和属性：

```typescript
globalLib({x: 1})
globalLib.version
globalLib.doSomething()
```



> 接下来是模块类库：

创建一个`module-lib.js`：

```javascript
//这是一个common.js的模块
const version = '1.0'

function doSomething() {
    console.log('moduleLib do something')
}

moduleLib.version = version
moduleLib.doSomething = doSomething

module.exports = moduleLib
```

我们在`index.ts`中引入：

```typescript
//报错，无法找到模块'./module-lib'的声明文件
const moduleLib = require('./module-lib')
```

我们创建声明文件`module-lib.d.ts`：

```typescript
//可以看到声明文件和全局的声明文件基本一致
declare function moduleLib(options: Options): void

//声明文件本身也是一个模块，所以不用担心接口暴露
interface Options {
    [key: string]: any
}

declare namespace moduleLib {
    const version: string
    function doSomething(): void
}
//最后用 export = 的形式导出
export = moduleLib
```

然后就可以看到`index.ts`中的报错已经消除，使用属性和方法也没有问题了。



> UMD库声明文件

先创建一个`umd-lib.js`：

```typescript
//一个 UMD模块
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(factory)
    } else if (typeof module === "object" && module.exports) {
        module.exports = factory()
    } else {
        root.umdLib = factory()
    }
}(this, function() {
    return {
        version: '1.0.0',
        doSomething() {
            console.log('umdLib do something')
        }
    }
}))
```

然后在`index.ts`中引用：

```typescript
//报错，无法找到模块'./umd-lib'的声明文件
import umdLib from './umd-lib'
```

编写声明文件`umd-lib.d.ts`：

```typescript
declare namespace umdLib {
    const version: string
    function doSomething(): void
}
//如果是UMD库，这条语句是不可缺少的
export as namespace umdLib
    
export = umdLib
```

发现已经可以正常使用。



##### 插件

> 模块化插件

有时候我们想给一个类库添加一些自定义的方法，以`moment`作为例子，首先安装：

```shell
npm i moment
```

然后在`index.ts`中导入：

```typescript
import moment from 'moment'

//给moment添加自定义的方法
//报错，类型 typeof moment 上不存在属性 myFunction
moment.myFunction = () => {}
```

我们可以使用`declare`来解决这个问题：

```typescript
import moment from 'moment'
//这里我们deaclare一个module，名为moment
declare module 'moment' {
    //使用export导出自定义的方法
    export function myFunction(): void
}

moment.myFunction = () => {}
```

> 全局插件

如果我们需要给全局变量添加一些方法，假如我们要给上面用到的`globalLib`增加一些自定义的方法，我们可以这样：

```typescript
//我们declare一个global
declare global {
    //里面定义一个命名空间，名称为要添加的类库名称
    namespace globalLib {
        //里面写要增加的方法
        function doAnything(): void
    }
}
//这时我们已经可以给globalLib增加一个doAnything方法
globalLib.doAnything = () => {}
```

这样其实对全局的命名空间造成了一定的污染，一般不建议这样做。

#### 声明文件的依赖

如果一个类库很大，那么它的声明文件也会很长，一般会按照模块进行划分，那么声明文件之间就会存在一些依赖关系。我们以`jQuery`为例子，看看它是怎么组织声明文件的：

首先查看`node_modules`下`@types`下的`jQuery`，找到`package.json`文件，可以发现字段`"types": "index"`，这表示所有声明文件的入口为`index.d.ts`，然后我们查看`index.d.ts`，代码如下：

```typescript
/// <reference types="sizzle" />
/// <reference path="JQueryStatic.d.ts" />
/// <reference path="JQuery.d.ts" />
/// <reference path="misc.d.ts" />
/// <reference path="legacy.d.ts" />

export = jQuery;
```

这里就是使用`三斜线`引入了其他的声明文件，这里声明文件分为两种：

- 一种是模块依赖，使用`types`属性，在这里的含义是`TypeScript`会在`@types`目录下寻找指定的模块，这里为`sizzle(jQuery的CSS选择器引擎)`，寻找到之后会将`sizzle`的声明文件引入进来。
- 另一种是路径依赖，使用`path`属性，这里是相对路径，也就是跟`index.d.ts`同级的声明文件，将它们引进来。

最后使用`export`进行导出。

如果现在编写声明文件还感觉很困难，可以参考官网的例子或者阅读知名类库的声明文件来学习声明文件是如何编写的。



#### tsconfig.json的配置

##### 文件相关

如果`tsconfig.json`没有任何配置，那么编译器就会根据默认的配置编译当前目录下的所有`ts`文件，`ts`文件有三种类型，分别是`.ts`、`.d.ts`、`.tsx`。

我们先在`src`文件夹下创建一个`a.ts`文件：

```typescript
let s: string = 'a'
```

> files

如果我们现在使用`tsc`命令进行编译，会将所有`ts`文件都进行编译，要编译单个文件，我们可以进行配置，打开`tsconfig.json`：

```json
{
    "files": [
        "src/a.ts"
    ]
}
```

`files`是一个数组，里面配置我们需要编译的文件路径，这里我们配置了`src`下的`a.ts`，这时我们在命令行运行`tsc`，这时将会只编译`a.ts`。

> include

如果要编译某个目录下所有的`ts`文件，可以配置`include`：

```json
{
    "include": [
        "src"
    ]
}
```

这将会编译`src`目录下所有的文件，包括子目录的文件。`include`是支持通配符的，如果我们这样配置：

```json
//将会只编译`src`目录下第一级目录下的文件，而不包括子目录
{
    "include": [
        "src/*"
    ]
}

//只会编译`src`二级目录下的文件
{
    "include": [
        "src/*/*"
    ]
}
```

> exclude

如果要排除某些文件（默认会排除`node_modules`也会排除所有的声明文件，所以不用单独配置），可以使用`exclude`：

```json
//src下的lib文件夹下的文件将不会编译
{
    "exclude": [
        "src/lib"
    ]
}
```

> 继承

**配置文件之间是可以继承的，可以把基础配置分离出来，方便复用。**

我们新建一个`tsconfig.base.json`，然后将`tsconfig.json`中的基础配置剪切过来：

```json
// tsconfig.base.json
{
    "files": [
        "src/a.ts"
    ],
    "include": [
        "src"
    ],
    "exclude": [
        "src/lib"
    ]
}
```

然后在`tsconfig.json`中可以通过`extends`配置型进行基础的配置：

```json
// tsconfig.json
{
    "extends": "./tsconfig.base"
}
```

在`tsconfig.json`中可以覆盖继承的配置，比如：

```json
// tsconfig.json
{
    "extends": "./tsconfig.base",
    //这里会覆盖继承的配置，不会忽略任何目录和文件
    "exclude": []
}
```

> compileOnSave

还有一个选项就是`compileOnSave`，这个选项会在保存文件时让编译器自动编译，但是`VSCode`目前还不支持这个配置，`Visual Studio 2015`安装`TypeScript 1.8.4及更高版本`以及[atom-typescript](https://github.com/TypeStrong/atom-typescript#compile-on-save)插件当前支持此功能。

##### 编译相关

跟编译相关的选项多达近百，所以这里只列举一些常用的选项，这些编译选项已经足够日常的开发需求，不常用的选项可以自行查询文档。

> incremental

这个配置是增量编译，`ts`编译器可以在第一次编译后生成一个可以存储编译信息的文件，在二次编译时会根据这个文件做增量的编译，从而提高编译的速度。

打开这个选项编译会生成一个文件`tsconfig.tsbuildinfo`，这个就是存储编译信息的文件。

> tsBuildInfoFile

可以指定编译信息文件的位置和名称，比如`"tsBuildInfoFile": "./bundleFile"`，可以将编译信息文件存储到当前文件夹下的`bundleFile`中。

> diagnostics

这个选项可以打印诊断信息，打开这个选项，在编译时打印诊断信息，比如编译时间。

> target

编译生成的`JavaScript`代码版本，比如`"target": "es5"`就是将`TypeScript`编译为`ES5`。

> module

编译生成的`JavaScript`代码所使用的模块标准，比如`"module": "commonjs"`即为`commonjs`。

> outFile

将多个相互依赖的文件生成一个文件，可以用在`AMD`模块中。比如`"outFile": "./app.js"`就会将依赖的文件打包成一个文件`app.js`。

> lib

`TypeScript`需要引用的一些类库，即声明文件。如果不指定，它会自己导入一些类库，假如我妈们设置`target`为`es5`，默认会导入`"lib": ["dom", "ese5", "scripthost"]`这三个库。

比如我们想使用一些`es2019`的特性：

```typescript
//flat是es2019的特性，直接使用会报错
//类型 (number | number[])[]上不存在属性flat
console.log([1,2,[3,4]].flat())
```

这时我们在配置文件中配置：

```json
"lib": ["dom", "es5", "scripthost", "es2019.array"]
```

这时就没有报错了。

> allowJs

开启这个选项会允许编译`JS`文件（JS、JSX）。

> checkJs

开启这个选项会允许在`JS`文件中报错，通常与`allowJs`配合使用。

> outDir

指定编译文件的输出目录，比如`"outDir": "./out"`。

> rootDir

指定输入文件目录（用于控制输出目录结构），默认为当前目录，比如`"rootDir": "./src"`。

>declaration

开启这个选项会自动帮助我们为编译文件生成一个声明文件。

> declarationDir

用于指定自动生成声明文件的路径。比如`"declarationDir": "./d"`，会将自动生成的声明文件放到当前文件夹的`d`目录之下。

> emitDeclarationOnly

开启这个选项只生成声明文件而不生成`JS`文件。

> sourceMap

开启这个选项会自动生成目标文件的`sourceMap`文件。

> inlineSourceMap

开启这个选项会生成目标文件的`inline sourceMap`（会包含在生成的`JS`文件之中）。

> declarationMap

开启这个选项会为声明文件也生成`sourceMap`。

> typeRoots

指定声明文件的目录，默认编译器会查找`node_modules/@types`下的所有声明文件。

> types

指定需要加载的声明文件的包，会从`node_modules/@types`查找，如果我们在这里只指定了某一个包，那么就只会加载指定包的声明文件，而不是`@types`下的全部。

> removeComments

开启这个选项会删除注释。

> noEmit

开启则不输出任何文件，什么都不会做。

> noEmitOnError

当发生错误时不输出任何文件。

> noEmitHelpers

这个选项涉及类的继承，当我们编写类继承并编译时，生成的`JS`文件除了我们自己的代码，还引入了一些其他的工具库函数`（helpers）`，副作用是会增大我们的编译文件体积，开启这个选项就不会生成`helper`函数。但是这时生成代码 中的`__extends`会是未定义的，需要配合下面的`importHelpers`选项使用。

> importHelpers

开启这个选项会通过`tslib`引入`helper`函数，文件必须是模块，否则无效。

> downlevelIteration

开启后，如果我们的语言是`es3/5`，就会对遍历器有比较低级的实现。

> strict

开启后将进行严格的类型检查，如果这里设置为`true`，那么以下配置项将全部为`true`：`alwaysStrict`、`noImplicitAny`、`strictNullChecks`、`strictFunctionTypes`、`strictProptyInitialization`、`strictBindCallApply`、`noImplicitThis`。

> alwaysStrict

在代码中注入`“use strict”`。

> noImplicitAny

不允许隐式的`Any`类型。

> strictNullChecks

不允许把`null`、`undefined`赋值给其他类型的变量。

> strictFunctionTypes

不允许函数参数双向协变（更严格的函数参数检查）。

> strictBindCallApply

进行严格的`bind`、`call`、`apply`检查，即使在使用`bind`、`call`、`apply`时，也进行严格的类型检查。

> noImplicitThis

不允许`this`有隐式的`Any`类型。因为`this`可能会为`undefined`，举个例子：

```typescript
class A {
    a: number = 1
    getA() {
        return function() {
            //如果开启则会报错，不开启不会报错
            console.log(this.a)
        }
    }
}

 const a = new A().getA()
 //如果没哟开启，上面不报错，但是运行会报错
 //会报错 undefined上没有属性a
 //因为此时的作用域已经改变，为全局作用域
 a()

//解决方法可以将getA的返回值改成箭头函数
```

开启这个选项可以避免这种因为作用域改变而导致的`this`指向问题。

> noUnusedLocals

检查只声明，但未使用的局部变量。

> noUnusedParameters

检查未使用的函数参数。

> noFallthroughCasesInSwitch

防止`switch`语句贯穿（即没有写`break`的情况）。

> noImplicitReturns

每个分支都要有返回值。

> esModuleInterop

允许使用`export = `导出时，由`import from`语法导入。

> allowUmdGlobalAccess

允许在模块中访问`UMD`全局变量。

> moduleResolution

模块解析策略，默认为`"moduleResolution": "node"`，还有一个解析策略为`classic`：

- `classic`:  用于`AMD`、`System`、`ES2015`，相对导入时会从相对文件夹下的`ts`，`d.ts`文件依次解析，非相对导入时会依次从里向外的`node_modules`目录中进行查找解析。
- `node`： 相对导入时，会依次解析相对文件夹下的`ts`、`tsx`、`d.ts`，如果没有找到，则会查找`package.json`，查看是否有`types`，如果没有，则会默认去查找`index.ts`、`index.tsx`、`index.d.ts`。非相对导入时会首先查找当前目录下的`node_modules`目录，依次解析`ts`、`tsx`、`d.ts`，没有则查看`package.json`，策略和相对导入相同，如果当前目录都没找到则会从里至外依照这个策略查找，直到根目录。

> baseUrl

解析非相对模块的基地址，默认为当前目录。

> paths

路径映射，相对于`baseUrl`。

比如我们不想引入`jQuery`的默认版本，而是想引入他的精简版本，可以这样写：

```json
"paths": {
    "jquery": ["node_modules/jquery/dist/jquery.slim.min.js"]
}
```

> rootDirs

将多个目录放在一个虚拟目录下，用于运行时。比如`"rootDirs": ["src", "out"]`，这会让`src`和`root`同处一个虚拟目录之中，让编译器认为他们同属一个目录之下。

> listEmittedFiles

编译完成后打印输出的文件列表。

> listFiles

编译完成后打印编译的文件列表（包括引用的声明文件）。

##### 工程引用

详见[工程引用](https://www.tslang.cn/docs/handbook/project-references.html)。

#### 编译工具

在之前我们使用`ts-loader`将`TypeScript`编译成`JavaScript`，`ts-loader`在内部调用了`TypeScript`的官方编译器`tsc`，所以`ts-loader`和`tsc`是共享`tsconfig.json`配置文件的。`ts-loader`也有一些自己的配置，我们打开之前编写的`webpack.base.config.js`，找到这一段：

```javascript
// webpack.base.config.js

module: {
        rules: [{
            test: /\.tsx?$/i,
            use: [{
                loader: 'ts-loader',
                //通过options来配置属性
                options: {
                    
                }
            }],
            exclude: /node_modules/
        }]
    }
```




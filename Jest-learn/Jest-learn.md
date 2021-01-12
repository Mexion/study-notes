## 一个简单的Jest例子

 首先写一个简单的`Jest`例子：

```js
test("测试加法 6 + 4", () => {
    expect(add(6, 4)).toBe(10);
})
```

这是一个简单的测试用例，测试目标是`add函数`。可以看到，一个`test函数`代表一个测试用例，`test`的第一个参数用来描述这个测试用例，第二个参数是一个函数，函数里写具体的测试代码。

`expect`是期待的意思，它接收测试函数，`expect(add(6, 4)).toBe(10)`比较语义化，很容易理解，期待`add(6, 4)`的返回结果是`10`。

显然，`expect`是一个可以链式调用的函数，它的返回值是一个对象，里面有`toBe`这个函数，测试函数的返回值会和`toBe`的参数进行比较。

当我们运行`jest`命令时，`Jest`会自动运行`.test.js`或者`.test.ts`后缀的文件。

## Jest的简单配置

使用命令 `npx jest --init`可以生成配置文件，根据提示选择即可。生成的`jest.config.js或者jest.config.ts`中的`coverageDirectory`配置项可以配置测试报告地址，使用命令`npx jest --coverage`可以生成测试报告。

使用命令`npx jest`进行测试，当要监听所有测试文件修改时可添加`--watchAll`，也就是说如果使用`npx jest --watchAll`，只要测试用例文件发生了变化，就会重新进行测试。

要使用`ESModule`进行模块导入导出可以使用`babel`，安装`@babel/core`和`@babel/preset-env`，要使用`typescript`需要安装`@babel/preset-typescript`、`babel-jest`、`@types/jest`、`ts-node`，根目录新建`babel.config.js`配置文件，配置如下：

```js
module.exports = {
  presets: [
    ['@babel/preset-env', {targets: {node: 'current'}}],
    '@babel/preset-typescript',
  ],
};
```

## Jest中的匹配器

在使用`Jest`的例子中，`toBe`是`Jest`的一个`匹配器`，它要求测试返回值要与期望值`完全相等`。

一些常用的匹配器如下：

| **匹配器**               | 用法                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `toBe`                   | `toBe`调用`Object.is`来比较值，要求测试返回值要与期望值`完全相等`，类似于使用`===`。 |
| `toEqual`                | `toEqual`递归比较对象实例的所有属性，只匹配内容不匹配引用。  |
| `toBeNull`               | `toBeNull`测试结果是否为`null`，严格等于`null`才可以，不能是`undefined`或者空字符串。 |
| `toBeUndefined`          | `toBeUndefined`测试结果是否为`undefined`，严格等于`undefined`才可以，不能是`null`或者空字符串。 |
| `toBeDefined`            | `toBeDefined`测试结果是否已经定义。                          |
| `toBeTruthy`             | `toBeTruthy`测试结果是否为真值。                             |
| `toBeFalsy`              | `toBeFalsy`测试结果是否为假值。                              |
| `not`                    | `not`是一个特殊的匹配器，它放在其他匹配器之前表示取反，比如`.not.toBeFalsy()`表示不是假值，`.not.toBe(10)`表示不等于`10`。 |
| `toBeGreaterThan`        | `toBeGreaterThan`只能接收`number | bigint`，表示测试结果要大于预期值。 |
| `toBeLessThan`           | `toBeLessThan`只能接收`number | bigint`，表示测试结果要小于预期值。 |
| `toBeGreaterThanOrEqual` | `toBeGreaterThanOrEqual`只能接收`number | bigint`， 表示测试结果要大于等于预期值。 |
| `toBeLessThanOrEqual`    | `toBeLessThanOrEqual`只能接收`number | bigint`， 表示测试结果要小于等于预期值。 |
| `toBeCloseTo`            | `toBeCloseTo`只能接收`number`，表示测试结果接近预期值。它可以接收第二个参数限制小数点后要检查的位数。默认值`2`，即`Math.abs(expected - received) < 0.005`。这个匹配器可以解决计算浮点数的精度问题，比如著名的`0.1+0.2`，期望值是`0.3`，用`toEqual`不能通过测试，会有精度问题。 |
| `toBeNaN`                | `toBeNaN`测试结果是不是`NaN`。                               |
| `toMatch`                | `toMatch`可以接收`regexp | string`，当接收的是字符串时，测试结果是否**包含**期望字符串，当接收一个正则时，会与表达式匹配。 |
| `toContain`              | `toContain`测试结果是否在预期数组或`Set`中，还可以测试结果字符串是否是预期字符串的子串。 |
| `toThrow`                | `toThrow`测试目标函数是否抛出异常，`toThrow`可以接收参数，只有抛出的异常是参数这个异常才可以通过，比如`toThrow("this is an error")`表示抛出的异常为`this is an error`才可以通过，参数也可以写正则表达式。如果测试不抛出异常，可以使用`.not.toThrow`。 |
| `toMatchObject`          | `toMatchObject`测试预期对象是不是结果的子集。                |
| `resolves`               | `resolves`不是一个函数，它用来拿到`Promise`的`resolve`数据再进行处理，比如`expect(Promise.resolve('lemon')).resolves.toBe('lemon')`，`Promise.resolve`返回的并不是真实数据，使用`resolves`链式调用判断即可。 |
| `rejects`                | `rejects`不是一个函数，使用方式和`resolves`一样，但是它是拿到`Promise`的`reject`数据再进行处理。 |

更多匹配器可以查看[Jest Expect](https://jestjs.io/docs/zh-Hans/expect)。


## Jest命令行工具的使用

当我们使用`npx jest --watchAll`时，`Jest`会一直监控测试用例文件的变化，我们在命令行中也可以看到如下几行提示：

```bash
Watch Usage
 › Press f to run only failed tests.
 › Press o to only run tests related to changed files.
 › Press p to filter by a filename regex pattern.
 › Press t to filter by a test name regex pattern.
 › Press q to quit watch mode.
 › Press Enter to trigger a test run.
```

我们在此时根据提示按下对应的按键就可以进行不同的操作，对应如下：

- `f`，只测试没有通过的测试用例，在此模式下再次按下`f`则退出这个模式。
- `o`，只测试改变了的测试文件，这个模式要结合`git`使用，与`git`仓库的代码进行比对。也可以在运行`Jest`测试时使用`jest --watch`而不是`--watchAll`，这会默认进入`o`模式。
- `p`，会根据正则来测试匹配文件名的测试用例，进入这个模式时会要求你填写正则，填写完成后按回车即可。
- `t`，会根据正则来测试匹配测试用例描述的测试用例，进入这个模式时会要求你填写正则，填写完成后按回车即可。
- `q`，退出测试。
- `enter`，触发测试。

## 异步代码的测试

如果要测试异步函数，按照普通的测试方式是不行的，不管怎样都会测试通过，因为函数是异步的，在`expect`还未执行时整个函数就已经走完了。解决方法是：`test`的回调函数中可以写一个参数`done`，只有在异步操作后调用`done`函数才会认为是测试结束。

```typescript
const fetchData = (cb) => {
    axios.get("http://www.example.com/example.json").then((response) => {
        // 这个函数是异步执行的
        fn(response.data);
    });
};

test("fetchData 返回结果为{ success: true }", (done) => {
    fetchData((data) => {
        expect(data).toEqual({
            sucess: true
        });
        // 执行一下done
        done();
    });
});
```



如果测试函数的返回结果是一个`promise`，此时不用`done`：

```typescript
const fetchData = () => {
    return axios.get("http://www.example.com/example.json");
}

test("fetchData 返回结果为{ success: true }", () => {
    // 要return
    return fetchData().then((response) => {
        expect(response.data).toEqual({
            success: true
        });
    });
});
```

假如我们需要返回结果为`404`，如果这样写：

```typescript
const fetchData = () => {
    return axios.get("http://www.example.com/example.json");
}

test("fetchData 返回结果为404", () => {
    // 要return
    return fetchData().catch((e) => {
        expect(e.toString().indexOf("404") > -1).toBe(true);
    });
});
```

这样写如果可以请求成功，那么`catch`就不会执行，`expect`也就不会执行。如果我们需要这个测试用例里`expect`至少执行一次，可以改成这样：

```typescript
test("fetchData 返回结果为404", () => {
    // assertions的参数为需要expect的调用次数
    expect.assertions(1);
    return fetchData().catch((e) => {
        expect(e.toString().indexOf("404") > -1).toBe(true);
    });
});
```

`expect.assertion(1)`表明后面至少要执行一次`expect`。

当返回结果是一个`Promise`时，我们也可以使用`resolves`和`rejects`进行处理： 
```typescript
// 拿到resolve的值进行判断
test("fetchData 返回结果为{ success: true }", () => {
    // 这里用toMatchObject和外层套了data是因为axios返回的结果中data里才是实际值
    return fetchData().resolves.toMatchObject({
        data: {
        success: true
    	}
    });
});

//拿到reject的值进行判断
test("fetchData 返回结果为404", () => {
    // 用toThrow是因为axios的返回结果reject的是Error
    return fetchData().rejects.toThrow();
});
```

我们也可以使用`async await`的语法，因为`async await`会等待后面的表达式运行结束，所以就不需要`return`了：

```typescript
// 拿到resolve的值进行判断
test("fetchData 返回结果为{ success: true }", async () => {
    await fetchData().resolves.toMatchObject({
        data: {
        success: true
    	}
    });
});

//拿到reject的值进行判断
test("fetchData 返回结果为404", async () => {
    await fetchData().rejects.toThrow();
});
```

那么既然使用`async await`了，我们直接等待异步运行结束拿到结果再进行判断即可，所以可以改成如下：

```typescript
// 拿到resolve的值进行判断
test("fetchData 返回结果为{ success: true }", async () => {
    const response = await fetchData();
    expect(response.data).toEqual({
        success: true
    })
});

//拿到reject的值进行判断
test("fetchData 返回结果为404", async () => {
    // 同样，因为要确保进入catch中，所以要assertions
    expect.assertions(1);
    // 使用 async await 捕获错误只能用try catch
    try {
        await fetchData();
    } catch(e) {
        expect(e.toString()).toEqual("Error: Request failed with status code 404"));
    }
});
```

## Jest中的钩子函数

如果要在测试用例之前做某些初始化或者准备，一定要将这些过程写在钩子函数中，否则可能会出现不符合预期的结果。

### beforeAll

`beforeAll`这个钩子函数会在所有测试用例开始之前执行。

### afterAll

`afterAll`这个钩子函数会在所有测试用例结束之后执行。

### beforeEach

`beforeEach`这个钩子函数会在每一个测试用例开始之前执行。

### afterEach

`afterEach`这个钩子函数会在每一个测试用例结束之后执行。

## describe分组和作用域

`Jest`中几个测试用例可以进行分组，将不同功能分成不同的组会更优雅美观。`describe`函数就是用来分组的，它接收两个参数，第一个参数为描述，第二个参数是一个函数，我们可以将同一组的测试用例以及钩子函数全放到里面：

```typescript
class Counter {
    constructor() {
        this.number = 0;
    }
    
    addOne() {
        this.number += 1;
    }
    
    addTwo() {
        this.number +=2;
    }
    
    minusOne() {
        this.number -= 1;
    }
    
    minusTwo() {
        this.number -=2;
    }

 
describe('测试Counter', () => {
    let counter = null;
    beforeEach(() => {
        counter = new Counter();
    });
 
    describe('测试增加', () => {
        test('测试 Counter 中 addOne 方法', () => {
            counter.addOne();
            expect(counter.number).toBe(1);
        });
        test('测试 Counter 中 addTwo 方法', () => {
            counter.addTwo();
            expect(counter.number).toBe(2);
        });
    });
 
    describe('测试减少', () => {
        test('测试 Counter 中 minusOne 方法', () => {
            counter.minusOne();
            expect(counter.number).toBe(-1);
        });
        test('测试 Counter 中 minusTwo 方法', () => {
            counter.minusTwo();
            expect(counter.number).toBe(-2);
        });
    });
});
```



 `describe`的回调函数中也可以嵌套`describe`，每一个`describe`都是一个作用域，也就是说每一个`describe`中都可以写自己的钩子函数，钩子函数的生效范围为它所属的`describe`下的所有测试用例（包括`子describe`下的测试用例），相同的钩子函数会先执行外层的再执行内层的。

## test.only只测试一个测试用例

我们在调试时如果只想查看某一个测试用例的结果，可以将`test`改成`test.only()`，那么这个测试文件中就只会执行这一个测试用例：

```typescript
test.only('测试 Counter 中 addOne 方法', () => {
            counter.addOne();
            expect(counter.number).toBe(1);
        });
```

## Jest中的mock

 现在有这么一个场景，假设我们需要测试这个函数:
```typescript
const runCallback = (cb: (...arg: any[]) => any) => {
  cb();
};
```
按照平时的测试方法，测试用例如下：

```typescript
test("测试runCallback", () => {
    const func = () => {
        return "hello";
    };
  expect(runcallback(func)).toBe("hello");
});
```

编写测试用例时我们要测试`cb`的执行情况，但是很显然`expect`是拿不到`cb`的结果的，因为`runCallback`里并没有返回`cb`的结果，而我们又不想因为测试违反我们本意地去修改我们的代码，此时`mock`就能解决这个问题。
我们可以写这样一个测试用例:

```typescript
test("测试runCallback", () => {
  // 我们调用jest.fn()生成一个mock func
  const func = jest.fn();
  // 将mock func传入测试函数中
  runCallback(func);
  //然后用toBeCalled判断这个函数是否被调用过
  expect(func).toBeCalled();
});
```
我们使用普通的函数是没法用`toBeCalled`来判断的，因为普通函数是无法追溯的。

`func`上挂在了一些属性，比如`func.mock`，我们打印一下`func.mock`，可以得到：

```typescript
{
      calls: [ [] ],
      instances: [ undefined ],
      invocationCallOrder: [ 1 ],
      results: [ { type: 'return', value: undefined } ]
    }
```

`calls`表示这个`mock func`被调用的情况，`instances`表示实例的`this`指向，`results`表示这个函数执行的输出结果。

`calls`是一个二维数组，内层数组表示函数被调用时的参数，这个函数被调用了几次就会有几个内层数组，这里调用了一次，没有传参，所以有一个内层数组，数组内为空。

假如我们改成这样：

```typescript
const runCallback = (cb: (...arg: any[]) => any) => {
  // 传参
  cb("param");
};

test("callback", () => {
  // jest.fn可以传入一个函数， 函数的返回值可以当做mock func的返回值
  const func = jest.fn(() => {
    // 可以写其他逻辑
    return "123";
  });
  // 也可以使用 func.mockImplementation(() => "123");
  // 这里只写了返回值，实际上可以在函数内部写其他逻辑
    
  // 调用了两次
  runCallback(func);
  runCallback(func);
  expect(func).toBeCalled();
  console.log(func.mock);
});

// 会输出：

{
      calls: [ [ 'param' ], [ 'param' ] ],      
      instances: [ undefined, undefined ],      
      invocationCallOrder: [ 1, 2 ],
      results: [
        { type: 'return', value: '123' },       
        { type: 'return', value: '123' }        
      ]
    }
```

我们要测试调用次数可以`expect(func.mock.calls.length).toBe(2)`，参数和返回值也类似，从`func.mock`上拿到后再匹配即可。

 除了像上面那样给`fn`传参来模拟返回值，也可以使用`mockReturnValue`和`mockReturnValueOnce`：

```typescript
test("callback", () => {
  // 这里我们不传参
  const func = jest.fn();
  // 使用 mockReturnValue
  func.mockReturnValue("a return value");
  runCallback(func);
  runCallback(func);
  expect(func).toBeCalled();
  console.log(func.mock);
});

// 会输出：

 {
      calls: [ [ 'param' ], [ 'param' ] ],
      instances: [ undefined, undefined ],
      invocationCallOrder: [ 1, 2 ],
      results: [
        { type: 'return', value: 'a return value' },
        { type: 'return', value: 'a return value' } 
      ]
    }
```

`mockReturnValueOnce`也一样，只不过它只模拟一次，每调用一次就模拟一次：

```typescript
test("callback", () => {
  // 这里我们不传参
  const func = jest.fn();
  // 使用 mockReturnValue
  func.mockReturnValueOnce("return value 1");
  // 也可以使用 func.mockImplementationOnce(() => "123");
    
  runCallback(func);
  runCallback(func);
  expect(func).toBeCalled();
  console.log(func.mock);
});

// 可得：

 {
      calls: [ [ 'param' ], [ 'param' ] ],
      instances: [ undefined, undefined ],
      invocationCallOrder: [ 1, 2 ],
      results: [
        { type: 'return', value: 'return value 1' },
        { type: 'return', value: undefined }        
      ]
    }
```

多次调用：

```typescript
test("callback", () => {
  const func = jest.fn();
  func.mockReturnValueOnce("return value 1");
  func.mockReturnValueOnce("return value 2");
  // 实际上 可以链式调用
  // func.mockReturnValueOnce("return value 1").mockReturnValueOnce("return value 2");
  runCallback(func);
  runCallback(func);
  expect(func).toBeCalled();
  console.log(func.mock);
});

// 可得：
{
      calls: [ [ 'param' ], [ 'param' ] ],
      instances: [ undefined, undefined ],
      invocationCallOrder: [ 1, 2 ],
      results: [
        { type: 'return', value: 'return value 1' },
        { type: 'return', value: 'return value 2' } 
      ]
    }
```

现在我们的`instances`里都是`undefined`，这是因为函数在当做普通函数调用时它的`this`指向在`node`中指向的是`undefined`，假如我们修改测试函数，去`new`这个`mock func`，此时`this`指向就会指向`mock func`作为构造函数构造的对象：

```typescript
const createObject = (classItem) => {
    // 去new这个构造函数
    new classItem();
};

test("测试createObject", () => {
    const func = jest.fn();
    createObject(func);
    console.log(func.mock);
});

// 可得
{
      calls: [ [] ],
      instances: [ mockConstructor {} ],
      invocationCallOrder: [ 1 ],
      results: [ { type: 'return', value: 'return value 1' } ]
    }
```

### mock异步请求

在测试真正的异步请求函数时，一般不会请求真正的接口，如果请求很多，那么测试需要花费很多时间，所以测试时我们只要求请求可以成功发送即可，而不关心返回的具体内容。

假如我们要使用`axios`进行请求，就可以使用`Jest`对`axios`进行模拟：

```typescript
import axios from "axios";

// 对axios进行模拟
jest.mock("axios");

// 测试函数
const fetchData = () => {
    return axios.get("/api").then(res => res.data);
};

// 显然axios上是不存在mockResolvedValue的，所以我们需要进行一个断言
const mockedAxios = axios as jest.Mocked<typeof axios>;
test("测试 fetchData", async () => {
  // 在axios发送get请求时，模拟返回数据
  mockedAxios.get.mockResolvedValue({
    data: "hello",
  });
  
  await fetchData().then((data) => {
      expect(data).toBe("hello");
  });
});
```

可以看到，上面使用了`mockResolveVAlue`来模拟成功请求返回的数据，同样的`api`还有`mockResolveVAlueOnce`模拟一次，`mockRejectedValue`、`mockRejectedValueOnce`模拟`reject`。

测试异步请求，除了去模拟`Axios`，还可以使用另外的方式，不去模拟`Axios`这个库，而是直接去模拟`fetchData`方法，`fetchData`里直接`resolve`或者`reject`请求内容。

假如测试函数写在项目目录的`fetchData.ts`中：

```typescript
import axios from "axios";

export const fetchData = () => {
    // 假如data的数据为 "(function(){return'123'})()"
    return axios.get("/api").then(res => res.data);
};
```

我们可以在同级目录新建`__mocks__`文件夹，文件夹中创建一个同名文件`fetchData.ts`，文件中同样写一个`fetchData`函数：

```typescript
export const fetchData = () => {
    return new Promise((resolve) => {
        resolve("(function(){return'123'})()");
    });
};
```

此时我们便可以写测试用例：

```typescript
import { fetchData } from "./fetchData";

// 我们不去mock axios 而是直接写路径， mock当前文件夹下的fetchData模块
jest.mock("./fetchData");

test("测试 fetchData", () => {
    // 当遇到fetchData模块中的内容，就用__mocks__下的替换
  return fetchData().then(data => {
    expect(eval(data)).toBe("123");
  });
});
```

但是这里会出现一个问题，假如`fetchData`还导出了另一个同步函数`getNumber`，并且我们不希望这个函数被模拟，如果直接测试肯定会报错，因为`__mocks__`下的模拟模块中并没有导出相同的`getNumber`函数，此时会报`undefined`。要解决这个问题可以这样：

```typescript
import { fetchData } from "./fetchData";
// 上面通过正常导入的是mock，下面才是导入真正的
// requireActual 很好理解 导入真正的
const { getNumber } = jest.requireActual("./fetchData");

test("测试getNumber", () => {
    expect(getNumber()).toBe(123);
})
```



### 取消mock

使用`mock`后，取消`mock`可以使用`jest.unmock`，比如`jest.unmock("axios")`或者`jest.unmock("./fetchData")`，此时就可以取消`mock`。

### 自动mock

如果我们使用了`__mocks__`文件夹的方式进行`mock`，可以更改配置文件`jest.config.ts`，将里面的`automock: false`改为`true`，更改后便不需要在测试文件中写`jest.mock`，当我们引入模块并使用模块中的内容时，`jest`会自动去当前目录中的`__mocks__`目录查找同名模块，查找到后会自动使用`mock`替换实际内容。

### mock 定时器

假如有这样一个函数：

```typescript
export const timerFunc = (cb: Function) => {
  setTimeout(() => {
    cb();
  }, 3000);
};
```

这是一个异步函数，所以测试用例如下：

```typescript
import { timerFunc } from "./timer";

test("测试 timerFunc", (done) => {
  timerFunc(() => {
    expect(1).toBe(1);
    done();
  });
});
```

运行测试用例，这个测试用例会在三秒之后测试结束，但是如果定时器的时间特别长或者有很多定时器，那么就会消耗大量的时间。为了避免这种情况，我们可以使用`mock timers`，测试用例修改如下：

```typescript
import { timerFunc } from "./timer";
// 我们先声明使用假的定时器
jest.useFakeTimers();
test("测试 timerFunc", () => {
  // 和之前一样，使用 mock func
  const fn = jest.fn();
  // 将mock func传入
  timerFunc(fn);
  // 调用所有的定时器
  jest.runAllTimers();
  // 我们期望这个函数被执行过一次
  expect(fn).toBeCalledTimes(1);
});
```

此时测试用例会在短暂的时间内测试通过。假如我们修改函数如下：

```typescript
export const timerFunc = (cb: Function) => {
  setTimeout(() => {
    cb();
    setTimeout(() => {
      cb();
    }, 3000);
  }, 3000);
};
```

假如我们使用`runAllTimers`，那么所有的定时器都会被执行，如果我们只想让外部的定时器执行，而不希望内部的定时器执行，可以将`runAllTimers`修改为`runOnlyPendingTimers`，即只运行挂载中的定时器，此时就会只运行外部的定时器，所以就没问题了。

`Jest 22`版本新增了一个更好用的`api`，叫`advanceTimersByTime`，修改测试用例：

```typescript
import { timerFunc } from "./timer";

jest.useFakeTimers();
test("测试 timerFunc", () => {
  const fn = jest.fn();
  timerFunc(fn);
  // 让时间快进 3s
  jest.advanceTimersByTime(3000);
  expect(fn).toBeCalledTimes(1);
});
```

因为我们快进了`3s`，所以只会运行`3s`的外部定时器，而内部的定时器应该在`6s`执行，自然不会执行，所以`toBeCalledTimes`为`1`，假如我们修改`advanceTimersByTime`的参数为`6000`，就会快进`6s`，内部的定时器也会执行，`toBeCalledTimes`也应该改为`2`才能通过。

当然，分多次执行也没问题：

```typescript
jest.useFakeTimers();
test("测试 timerFunc", () => {
  const fn = jest.fn();
  timerFunc(fn);
  // 先快进3s 执行一次
  jest.advanceTimersByTime(3000);
  expect(fn).toBeCalledTimes(1);
  // 再快进3s 执行两次
  jest.advanceTimersByTime(3000);
  expect(fn).toBeCalledTimes(2);
});
```

这也就出现了另一个问题，时间的快进是可以叠加的，如果有多个`test`，可能会造成多个测试用例之间的影响。此时可以使用`beforeEach`来解决，我们在每个测试用例开始前都重新调用一遍`useFakeTimers`对`timers`做一个初始化：

```typescript
beforeEach(() => {
  jest.useFakeTimers();
});

test("测试 timerFunc", () => {
  const fn = jest.fn();
  timerFunc(fn);
  jest.advanceTimersByTime(3000);
  expect(fn).toBeCalledTimes(1);
  jest.advanceTimersByTime(3000);
  expect(fn).toBeCalledTimes(2);
});
```

### 对类mock理解单元测试和集成测试

假如我们的`utils.ts`导出了这样一个类：

```typescript
class Util {

  a(a: any) {
	// 复杂逻辑
  }

  b(b: any) {
	// 复杂逻辑
  }
}

export default Util;
```

它有`a`、`b`两个方法，这两个方法内部的逻辑十分复杂，然后`demoFunction.ts`使用了这个类：

```typescript
import Util from "./util";

const demoFunction = (a: any, b: any) => {
  const util = new Util();
  util.a(a);
  util.b(b);
};

export default demoFunction;
```

现在我们想测试`demoFunction`，如果直接去运行`demoFunction`，因为里面运行的两个函数异常复杂，就会十分消耗性能，并且真实的`a`和`b`的执行对测试没有帮助，所以我们可以创造一个模拟的`Util`类，让这个模拟类里的`a`和`b`执行就可以了，我们可以写如下测试用例：

```typescript
import demoFunction from "./demo";
import Util from "./util";

// mock util
// jest 发现模拟的是一个类时，会自动把类的构造函数和所有方法变成jest.fn()
jest.mock("./util");

test("测试 demoFunction", () => {
  //我们调用一次测试函数，内部的Util会被替换成mock util
  demoFunction(1, 2);
  // 既然是模拟的，我们就可以追溯到是否执行
  expect(Util).toHaveBeenCalled();
  // 对应模拟实例里面的a和b是否执行
  expect((Util as jest.Mock).mock.instances[0].a).toHaveBeenCalled();
  expect((Util as jest.Mock).mock.instances[0].b).toHaveBeenCalled();
});
```

我们也可以对模拟进行定制，在目录的`__mocks__`目录中新建同名文件`util.ts`：

```typescript
// 我们可以对里面的构造函数和各个函数自定义
// 实际上我们不写这个文件，这就是jest帮助我们自动完成的过程
const Util = jest.fn(() => {
  // 在fn里自定义逻辑，比如简单的console
  console.log("util");
});

Util.prototype.a = jest.fn(() => {
  console.log("a");
});

Util.prototype.b = jest.fn(() => {
  console.log("b");
});

export default Util;
```

除了写在`__mocks__`里，也可以换一种形式：

```typescript
jest.mock("./util", () => {
    const Util = jest.fn(() => {
      console.log("util");
    });

    Util.prototype.a = jest.fn(() => {
      console.log("a");
    });

    Util.prototype.b = jest.fn(() => {
      console.log("b");
    });
    // return 出去
    return Util;
});
```

写在`mock`的第二个参数里和写在`__mocks__`同名文件里是一样的。



到目前我们所做的都是单元测试，所谓单元测试，就是只对当前单元的代码进行测试，就像测试`demoFunction`，我们只关注`demoFunction`这个函数的运行情况，而不关注外部引入的内容，比如`Util`，如果外部引入的东西会造成性能上的损耗，我们就可以用更简单的东西来模拟这些外部的东西，让我们这个单元测试更快，所以在单元测试里使用`mock`很常见，以提升单元测试的性能。

与单元测试不同的是集成测试，集成测试不仅对自己单元的内容做测试，对自己单元中引用的外部模块也一并做测试。

更多`mock-func api`可以查看[mock-function-api](https://jestjs.io/docs/en/mock-function-api)。

## Snapshot 快照测试

在项目中我们一般会有一些配置文件，比如：

```typescript
export const generateConfig  = () => {
    return {
        server: 'http://localhost',
        port: '8080'
    };
};
```

测试用例如下：

```typescript
import { generateConfig } from './config.ts';
 
test('测试 generateConfig', () => {
    expect(generateConfig()).toEqual({
        server: "http://localhost",
        port: "8080"
    });
});
```

此时可以测试通过，但是当我们的配置不断增加的时候，测试用例也需要同步更改，否则两边不一致就会导致测试用例无法通过。我们可以使用`toMatchSnapshot`来修改这个测试用例：

```typescript
test('测试 generateConfig', () => {
    expect(generateConfig()).toMatchSnapshot();
});
```

当第一次执行这个测试用例的时候，会在项目目录下生成`__snapshots__`文件夹，里面根据测试用例的结果生成快照，假设重新进行测试，会再次生成快照与之前的快照做匹配，假如两个快照相同，则测试通过，否则不通过。

此时如果要确认修改，可以刷新快照，在`watch`或者`watchAll`模式下可以按`w`显示命令行工具菜单，此时可以看到里面有`u`这个选项，按下`u`即可生成新的快照替换老的快照。当然，不在`watch`模式下也可以更新快照，使用`npx jest --updateSnapshot`或者`npx jest -u`命令即可。

假如有两个测试用例快照都失败，但是我们只想更新某一个，可以在`watch`或者`watchAll`模式下按`w`然后按`i`进入交互模式，根据提示来对每一个失败的快照进行操作，如果要通过就按`u`，跳过就按`s`，退出交互模式按`q`。

可以使用`--testNamePattern`或`-t`来只更新与正则匹配的测试用例，比如`npx jest -t=generateConfig`那么就只会测试描述中包含`generateConfig`的测试用例。

有时候，可能配置文件里并不是写死的，而是动态变化的，比如`new Date()`：

```typescript
export const generateConfig  = () => {
    return {
        server: 'http://localhost',
        port: '8080',
        time: new Date(),
    };
};
```

此时测试用例就会始终无法通过，因为两个时间不会相等。此时可以修改测试用例如下：

```typescript
test('测试 generateConfig', () => {
    // toMatchSnapshot里写对象，对象中的time可以是任何Date
    // 不必每次都相等
    expect(generateConfig()).toMatchSnapshot({
        time: expect.any(Date),
    });
});
```

当然，`any`里可以传任何类型，比如`Number`，`String`。

### 内联快照

以上生成的`Snapshot`会生成外部文件，除了这种方式，我们也可以使用`内联快照`。在使用`内联快照`之前，我们需要在项目中安装`prettier`，因为这个功能是依托于`prettier`的，使用`npm`或`yarn`安装即可。
安装好之后，我们把`toMatchSnapshot`更改为`toMatchInlineSnapshot`即可：
```typescript
test('测试 generateConfig', () => {
    expect(generateConfig()).toMatchInlineSnapshot({
        time: expect.any(Date),
    });
});
```
重新运行测试，会看到此测试用例会改成：
```typescript
test("测试 generateConfig", () => {
  expect(generateConfig()).toMatchInlineSnapshot(
    {
      time: expect.any(Date),
    },
    `
    Object {
      "domain": "localhost",
      "port": "8080",
      "server": "http://localhost",
      "time": Any<Date>,
    }
  `
  );
});
```

快照会以字符串模板的形式作为`toMatchInlineSnapshot`的第二个参数进行保存。

## DOM测试

在`Jest`中进行`DOM`测试非常简单，直接进行`DOM`操作即可，比如我们有一个`addDivToBody`函数，这个函数很简单，每调用一次就向`body`中插入一个`div`：

```typescript
export const addDivToBody = () => {
  const body = document.querySelector("body");
  body?.append(document.createElement("div"));
};
```

可以直接这样写测试用例：

```typescript
import { addDivToBody } from "./dom";

test.only("测试 addDivToBody", () => {
  addDivToBody();
  addDivToBody();
  expect(document.querySelector("body")?.querySelectorAll("div").length).toBe(2);
});
```

`Jest`实际的运行环境实际上是`node`，之所以可以直接操作`DOM`是因为它模拟了一套`DOM`的`api`，即`jsdom`。

## Vue中的TDD与单元测试

### 什么是TDD？

`TDD`即`Test Driven Development`，测试驱动开发，简单来说就是先写测试用例再写代码。

`TDD`的开发流程如下：

1. 编写测试用例。
2. 运行测试，测试用例无法通过测试。
3. 编写代码，使测试用例通过测试。
4. 优化代码，完成开发。
5. 重复上述步骤。

`TDD`的优势如下：

- 长期减少回归`bug`。
- 代码质量更好（组织、可维护性）。
- 测试覆盖率高。
- 错误测试代码不容易出现。

###  Vue环境中使用Jest

首先安装`Vue`脚手架：

```bash
yarn add @vue/cli -g
```

安装完成后生成项目：

```bash
vue create jest-vue
```

我们选择`Manually select features`自定义安装，根据需求选择配置项，多选时空格选择，回车下一步，在选择时我们除了自己的需求之外，还需要选择`Unit Testing`这个选项，在`Pick a unit testing solution`时选择`Jest`，创建完成后我们可以更改`package.json`下的`scripts`中的`"test:unit"`为`"vue-cli-service test:unit --watch"`开启监控模式，然后运行`yarn run test:unit`即可开启测试。

在根目录的`tests/unit`下有一个`example.spec.ts`，这是一个示例测试文件，我们可以通过这个文件来了解`jest`在`Vue`中的使用：

```typescript
// 首先从 test-utils中引入了shallowMount
import { shallowMount } from "@vue/test-utils";
// 需要测试 HelloWorld 这个组件
import HelloWorld from "@/components/HelloWorld.vue";

// describe 不用多说
describe("HelloWorld.vue", () => {
  // it 是 test 的别名
  it("renders props.msg when passed", () => {
    const msg = "new message";
    // shallowMount 即浅挂载 只挂载组件而忽略组件的子组件的挂载
    // 所以shallowMount适合单元测试使用
    // 与之相对的是 mount，会挂载组件以及子组件，适合集成测试使用
    // 通过 propsData向组件传递props
    // shallowMount 会返回 wrapper
    // wrapper 上挂载了一系列的方法，以便进行测试
    const wrapper = shallowMount(HelloWorld, {
      propsData: { msg }
    });
    // wrapper.text() 返回组件所有元素的文本内容
    // 所有元素的文本会拼接成一个字符串，toMatch会用正则匹配
    expect(wrapper.text()).toMatch(msg);
  });
});
```

`wrapper`上挂载了一系列方法，除了上面用到的`text`返回组件中所有元素的文本内容，还有`find`查找组件内的一个`DOM`元素，`findAll`查找多个元素，它们的用法和`querySelector`与`querySelectorAll`类似。`props`可以拿到对应参数`key`的`prop`。其它的更多方法可以访问[wrapper-methods](https://vue-test-utils.vuejs.org/v2/api/#wrapper-methods)。

### Vue组件的快照测试

在测试一个组件可以正常渲染，而先不测功能的时候，可以这样写测试用例：

```typescript
describe("HelloWorld.vue", () => {
  it("测试组件正常渲染", () => {
    const msg = "new message";
    const wrapper = shallowMount(HelloWorld, {
      propsData: { msg }
    });
    expect(wrapper).toMatchSnapshot();
  });
});
```

只要组件第一次可以渲染，`Jest`就会帮助我们生成快照如下：

```typescript
exports[`HelloWorld.vue 测试组件正常渲染 1`] = `
<div class="hello">
  <h1>abc</h1>
  <h1>new message</h1>
</div>
`;
```

快照的使用可以让我们及时捕获到`UI`的变化，如果组件在此后的渲染中，渲染结果与之不一致，则测试就无法通过，如果确定要更新快照，可以使用`-u`或者在监控模式下进行。


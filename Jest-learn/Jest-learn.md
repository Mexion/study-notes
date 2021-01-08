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
| `toMatch`                | `toMatch`可以接收`regexp | string`，当接收的是字符串时，测试结果是否包含期望字符串，当接收一个正则时，会与表达式匹配。 |
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

我们要测试调用次数可以`ecpect(func.mock.calls.length).toBe(2)`，参数和返回值也类似，从`func.mock`上拿到后再匹配即可。

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

更多`mock-func api`可以查看[mock-function-api](https://jestjs.io/docs/en/mock-function-api)。
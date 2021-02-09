# Nest.js介绍

 `Nest` 是一个渐进的 `Node.js` 框架，可以在 `TypeScript` 和 `JavaScript`之上构 建高效、可伸缩的企业级服务器端应用程序。 `Nest` 基于 `TypeScript` 编写并且结合了 `OOP（面向对象编程）`，`FP（函数式编程）`和` FRP （函数式响应编程）`的相关理念。在设计上的很多灵感来自于 `Angular`，`Angular` 的很多模 式又来自于 `Java` 中的 `Spring` 框架，依赖注入、面向切面编程等，所以我们也可以认为： `Nest` 是 `Node.js` 版的 `Spring` 框架。 `Nest` 框架底层 `HTTP` 平台默认是基于 `Express` 实现的，所以无需担心第三方库的缺失。 `Nest` 旨在成为一个与平台无关的框架。 通过平台，可以创建可重用的逻辑部件，开发人员 可以利用这些部件来跨越多种不同类型的应用程序。 从技术上讲，`Nest` 可以在创建适配器后使用任何 `Node` `HTTP` 框架。 有两个支持开箱即用的 `HTTP` 平台：`Express` 和 `Fastify`。 您 可以选择最适合您需求的产品。  

 `Nest.js` 的核心思想就是提供了一个层与层直接的耦合度极小,抽象化极高的一个架构 体系。  

# Nest.js创建项目

可以通过`Nest-cli`来创建项目，首先安装：

```bash
npm i -g @nestjs/cli
// 或者
yarn global add @nestjs/cli
```

安装完成后输入：

```bash
nest new nestdemo	// nestdemo是自己的项目名
```

根据提示选择`npm`或者`yarn`作为包管理器即可创建一个项目。

如果使用`yarn`进行安装，创建项目时出现`Error: Collection "@nestjs/schematics" cannot be resolved.`错误，可以安装：

```bash
yarn global add @nestjs/schematics
```

就可以解决这个问题。

在项目目录使用`yarn run start:dev`即可以开发模式启动，所有修改会立即热更新生效。

## 项目文件分析

`src`目录下的`main.ts`是项目的入口文件，它包含一个异步函数，用来引导整个应用程序 ：

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

// 定义了一个启动方法
async function bootstrap() {
  // 通过NestFactory.create来创建应用，传入模块AppModule
  // 每个 Nest 应用程序至少有一个模块，即根模块AppModule是根模块
  // 模块定义如何组装应用
  const app = await NestFactory.create(AppModule);
  // 在bootstrap中可以配置一些全局的中间件，切换平台等操作
  // 监听3000端口
  await app.listen(3000);
}
// 启动
bootstrap();
```

要创建`Nest`应用程序实例，我们使用核心`NestFactory`类。 `NestFactory`公开了一些允许创建应用程序实例的静态方法。 `create()`方法返回一个应用程序对象，该对象满足`INestApplication`接口。在上面的`main.ts`示例中，我们只需启动`HTTP`侦听器，即可让应用程序等待入站`HTTP`请求。

`app.module.ts`如下：

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
// 使用Module装饰器来装饰AppModule类
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

可以看到里面注册了`controller`和`service`。

`app.controller.ts`就是控制器文件，即`MVC`里的`C`：

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  // 这里是typescript的简便写法
  // 定义参数的同时还定义了类的属性
  constructor(private readonly appService: AppService) {}
// 注册了 / 路径，也就是http://localhost:3000/ 调用的是getHello这个方法
// 而它里面又去调用service里的getHello
  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

`app.service.ts`即`MVC`里的`M`，用来处理业务：

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    // 最终返回 Hello World!
    return 'Hello World!';
  }
}

```

# 概述

## 控制器

 控制器即`MVC`中的`Controller`， 它负责处理传入的请求, 并返回对客户端的响应。当接收到请求时，它负责解析请求并将请求信息传入`Service`层进行业务处理，待`Service`处理完成后再将信息返回给客户端。 

可以自己写控制器，也可以使用`Nest-Cli`帮助生成，使用`Cli`生成需要使用`nest g`命令（可以使用`nest g --help`查看所有命令），比如要生成一个`article`控制器，项目根目录下使用命令：

```bash
$ nest g controller article
```

可得：

```bash
CREATE src/article/article.controller.spec.ts (499 bytes)
CREATE src/article/article.controller.ts (103 bytes)
UPDATE src/app.module.ts (334 bytes)
```

可以看到会在`src`下创建`article`目录，并在底下创建`article.controller.spec.ts`测试文件；`article.controller.ts`控制器文件，并且更新了根模块文件`app.module.ts`，在`@Module`装饰器的对象参数中的`controllers`数组中增加`ArticleController`。

`article.controller.ts`文件如下：

```typescript
import { Controller } from '@nestjs/common';
// 可以看到就是创建了controller类，并且用Controller这个装饰器来装饰它
// 此时路径 http://localhost:3000/article都会由此controller控制
@Controller('article')
export class ArticleController {}
```

此时这个控制器内我们还没有写任何的请求以及路径，如果直接访问`http://localhost:3000/article`肯定返回的是`404`，假如使用`GET`请求，可以这样写：

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('article')
export class ArticleController {
  // 用装饰器Get来定义路径
  // 为空时即为根路径，这里是article/
  // 装饰的函数可以随意命名
  @Get()
  getArticle() {
    return 'article';
  }
}
```

### Get Post以及获取参数

此时这个控制器内我们还没有写任何的请求以及路径，如果直接访问`http://localhost:3000/article`肯定返回的是`404`，假如使用`GET`请求，可以这样写：

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('article')
export class ArticleController {
  // 用装饰器Get来定义路径
  // 为空时即为根路径，这里是article/
  // 装饰的函数可以随意命名
  @Get()
  getArticle() {
    return 'article';
  }
}
```

除了`Get`，还有`Post`：

```typescript
import { Controller, Post } from '@nestjs/common';

@Controller('article')
export class ArticleController {

  @Post()
  createArticle() {
    return 'ok';
  }
}
```

 `Nest.js` 也提供了其他 `HTTP` 请求方法的装饰器 `@Put()` 、`@Delete()`、`@Patch()`、 `@Options()`、 `@Head()`和 `@All()`。

### 获取请求参数装饰器

 在 `Nest.js` 中获取提交的数据可以使用装饰器来获取。 

| 装饰器                  | 内容                            |
| ----------------------- | ------------------------------- |
| @Request()              | req                             |
| @Response()             | res                             |
| @Next()                 | next                            |
| @Session()              | req.session                     |
| @Param(key?: string)    | req.params / req.params[key]    |
| @Body(key?: string)     | req.body / req.body[key]        |
| @Query(key?: string)    | req.query / req.query[key]      |
| @Headers(name?: string) | req.headers / req.headers[name] |
| @Ip()                   | req.ip                          |

 为了与底层 `HTTP`平台(如 `Express`和 `Fastify`)之间的类型兼容，`Nest` 提供了 `@Res()`和 `@Response()` 装饰器。`@Res()`只是 `@Response()`的别名。两者都直接公开底层响应对象接口。在使用它们时，您还应该导入底层库的类型(例如：`@types/express`)以充分利用它们。注意，在方法处理程序中注入 `@Res()`或 `@Response()` 时，将 `Nest`置于该处理程序的特定于库的模式中，并负责管理响应。这样做时，必须通过调用响应对象(例如，`res.json(…)`或 `res.send(…)`)发出某种响应，否则`HTTP`服务器将挂起。 如果想把响应处理逻辑留给框架，需要将`passthrough`参数设置为`true` ，这样就可以直接返回响应。

要获取`query string`，可以这样写：

```typescript
import { Controller, Get, Post, Query } from '@nestjs/common';

@Controller('article')
export class ArticleController {
  @Get()
  getArticle(@Query() query) {
    // 假如访问 article?id=123&name=abc
    // 可得 { id: '123', name: 'abc' }
    console.log(query);
    return query;
  }
}
```

也可以通过`@Request`装饰器拿到`request`再从`request`中取出`query`：

```typescript
// 下面这样写也可以
@Get()
  getArticle(@Request() request) {
    const { query } = request;
    console.log(query);
    return query;
  }
```

如果只想获取某一个`query string`，可以在装饰器中写`key`，比如：

```typescript
@Get()
  // 只获取id
  getArticle(@Query('id') id) {
    console.log(id);
    return id;
  }
```



要获取`Post`的`body`，在`Express`或者`Koa`中我们需要`body-parser`中间件，但在`Nest`中可以使用`@Body`装饰器来获取：

```typescript
@Post()
  createArticle(@Body() body) {
    return body;
  }
```

### 动态路由

 当需要接受动态数据作为请求的一部分时，例如`GET /cats/1`)获取`id`为`1`的`cat`，`id`是由客户端传过来的，是动态的。为了定义带参数的路由，我们可以在路由中添加路由参数标记，以捕获请求 `URL` 中该位置的动态值。下面示例中的路由参数标记演示了此用法。可以使用 `@Param()` 装饰器访问以这种方式声明的路由参数，该装饰器应添加到函数签名中。  

```typescript
// 路径前加 : 表名这是一个动态路由
// id 这个key的值是动态的
@Get(':id')
  getArticle(@Param() param) {
    // 访问 article/12
    // 可得 { id: '12' }
    console.log(param);
    return param;
  }
```

可以使用多个，比如：

```typescript
 @Get(':id/:name')
  getArticle(@Param() param) {
    // 访问 article/12/abc
    // 可得 { id: '12', name: 'abc' }
    console.log(param);
    return param;
  }
```

## 提供者

 `Providers` 是 `Nest` 的一个基本概念。许多基本的 `Nest` 类可能被视为 `provider `。比如 `service`,` repository`, `factory`, `helper` 等等。 他们都可以通过 `constructor` **注入**依赖关系。 这意味着对象可以彼此创建各种关系，并且“连接”对象实例的功能在很大程度上可以委托给 `Nest`运行时系统。 `Provider` 只是一个用 `@Injectable()` 装饰器注释的类。 

### 服务

 `服务（service）` 是一个`provider`，可以通过 `constructor` 注 入依赖关系。服务通过`@Injectable()` 装饰器来标记自己可被当做`Provider`注入到`Controller`的`constructor`中。

使用`Nest-cli`创建`service`可以使用命令`nest g service`，比如要创建一个`user service`：

```bash
nest g service user
```

完成后会自动生成`user`目录，并创建`user.service.spec.ts`、`user.service.ts`，并且更新根模块文件`app.module.ts`，在`@Module`装饰器的对象参数中的`providers`数组中增加`UserService`。

生成的`service`很简单，就是使用`@Injectable`装饰了`UserService`类：

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  // 我们简单写一个方法，返回一个字符串
  getUser() {
    // 在service里写各种业务，然后返回结果
    return 'This is a user';
  }
}
```

我们就可以在**任何**`controller`的构造函数中定义`UserService`的参数来让`Nest`注入`UserService`的实例：

```typescript
import { Controller, Get } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('user')
export class UserController {
  // 定义了UserService参数和类属性
  constructor(private readonly userService: UserService) {}

  // 定义了一个简单的请求接口，在里面调用service的getUser
  @Get()
  getUser() {
    return this.userService.getUser();
  }
}
```

此时运行`yarn run start:dev`，访问`http://localhost:3000/user`，即可得到返回`This is a user`。

### 提供者的作用域

`NestJs`提供了`@Scope()`装饰器来指定作用域：

| scope名称 | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| SINGLETON | 提供程序的单个实例在整个应用程序中共享。实例生存期与应用程序生命周期直接相关。应用程序启动后，所有单例提供程序都将实例化。默认情况下使用Singleton范围。 |
| REQUEST   | 为每个传入请求专门创建提供者的新实例。请求完成处理后，将垃圾回收该实例。 |
| TRANSIENT | 瞬态提供程序不跨消费者共享。每个注入临时提供程序的使用者都将收到一个新的专用实例，每次注入都会实例化。 |

```typescript
@Injectable({scope: Scope.REQUEST})
export class UserService {

}
```

对于大多数用例，建议使用单例作用域。在使用者和请求之间共享提供程序意味着在应用程序启动期间，实例可以被缓存并且其初始化仅发生一次。

### 基于属性注入

上文中的注入都是基于构造函数的，这样做有一个缺陷，如果涉及到继承的话，子类必须显示调用`super`来实例化父类。如果父类的构造函数参数过多的话反而成了子类的负担。

针对这个问题，可以`基于属性`进行注入。

```typescript
@Controller('users')
export class UserController {
    @Inject(标识)
    private readonly userService:UserService;
}
```

### 自定义提供者

#### 使用值

上文中提供的`Services`一般用在编写业务逻辑，结构基本是固定的，如果需要集成其他库作为注入对象的话，需要使用的自定义的服务提供者。

比如我们使用`sequelize`创建了数据库连接，想把它注入到我们的`Services`中进行数据库操作。可以使用以下方式进行处理：

```typescript
// sequelize.ts 数据库访问
export const sequelize = new Sequelize({
    // ...
});
```

```typescript
// sequelize.provider.ts
import {sequelize} from './sequelize';

export const sequelizeProvider = {
    provide: 'SEQUELIZE', // 服务提供者标识
    useValue: sequelize, // 直接使用值
}
```

```typescript
// user.module.ts
@Module({
    providers:[UserService, sequelizeProvider]
})
export class UserModule {}
```

```typescript
// user.service.ts
@Injectable()
export class UserService {
    // 使用@Inject(标识)注入
    constructor(@Inject('SEQUELIZE') private readonly sequelize: Sequelize) {}
}
```

#### 使用类

 `OOP`的一个重要思想就是`面向接口化`设计，比如我们开发了一个日志接口，有写入本地文件的实现，也有写入`syslog`的实现。依赖注入到时候我们希望使用接口进行注入，而不是具体的实现。 

```typescript
// logger.ts
export interface Logger {
    log(log:string);
}
```

```typescript
// file.logger.ts
export class FileLogger implements Logger {
    log(log:string) {
        // 写入本地文件
    }
}
```

```typescript
// syslog.logger.ts
export class SyslogLogger implements Logger {
    log(log:string) {
        // 写入Syslog
    }
}
```

```typescript
// logger.provider.ts
export const loggerProvider = {
    provide: Logger, // 使用接口标识
    useClass: process.env.NODE_ENV==='development'?FileLogger:SyslogLogger, // 开发日志写入本地，生产日志写入syslog
}
```

```typescript
// user.module.ts
@Module({
    providers:[UserService,loggerProvider]
})
export class UserModule {

}
```

```typescript
// user.service.ts
@Injectable()
export class UserService {
    constructor(@Inject(Logger) private readonly logger: Logger) {}
}
```

#### 使用工厂

工厂模式相信大家都不陌生，工厂模式本质上是一个函数或者方法，返回我们需要的产品。

传统的第三方库都是提供`callback`形式或者事件形式的进行连接，比如`redis`， 在与数据库的连接建立之前，您可能不希望开始接受请求。 在这种情况下你应该考虑使用异步 `provider`。 如果需要使用该类型的注入对象，工厂模式是最佳方式。

以下是使用工厂模式创建数据库连接的例子：

```typescript
// database.provider.ts
export const databaseProvider = {
    provide:'DATABASE',
    useFactory: async(optionsProvider: OptionsProvider) { // 使用依赖，注入顺序和下面定义的顺序一致
    	// 实际上也可以使用async await
        return new Promise((resolve, reject) => {
            const connection = createConnection(optionsProvider.getDatabaseConfig());
            connection.on('ready',()=>resolve(connection));
            connection.on('error',(e)=>reject(e));
        });
    },
    inject:[OptionsProvider], // 注入依赖
}
```

```typescript
// user.module.ts
@Module({
    providers:[OptionsProvider, databaseProvider]
})
export class UserModule {

}
```

```typescript
// user.service.ts
@Injectable()
export class UserService {
    constructor(@Inject('DATABASE') private readonly connection: Connection) {}
}
```

1. 工厂函数可以接受(可选)参数。
2. `inject` 属性接受一个提供者数组，在实例化过程中，`Nest` 将解析该数组并将其作为参数传递给工厂函数。这两个列表应该是相关的: `Nest` 将从 `inject` 列表中以相同的顺序将实例作为参数传递给工厂函数。

#### 别名提供者

`useExisting` 语法允许您为现有的提供程序创建别名。这将创建两种访问同一提供者的方法。在下面的示例中，(基于`string`)令牌 `'AliasedLoggerService'` 是(基于类的)令牌 `LoggerService` 的别名。假设我们有两个不同的依赖项，一个用于 `'AlilasedLoggerService'` ，另一个用于 `LoggerService` 。如果两个依赖项都用单例作用域指定，它们将解析为同一个实例。

```typescript
@Injectable()
class LoggerService {
  /* implementation details */
}

const loggerAliasProvider = {
  provide: 'AliasedLoggerService',
  useExisting: LoggerService,
};

@Module({
  providers: [LoggerService, loggerAliasProvider],
})
export class AppModule {}
```

#### 非服务提供者

虽然提供者经常提供服务，但他们并不限于这种用途。提供者可以提供任何值。例如，提供程序可以根据当前环境提供配置对象数组，如下所示:

```typescript
const configFactory = {
  provide: 'CONFIG',
  useFactory: () => {
    return process.env.NODE_ENV === 'development'
      ? devConfig
      : prodConfig;
  },
};

@Module({
  providers: [configFactory],
})
export class AppModule {}
```

## 模块

 模块是具有 `@Module()` 装饰器的类。 `@Module()` 装饰器提供了元数据，`Nest` 用它来组织应用程序结构。 

 每个 `Nest` 应用程序至少有一个模块，即根模块。根模块是 `Nest` 开始安排应用程序树的地方。 

`@module()` 装饰器接收一个描述模块属性的对象：

|             |                                                            |
| ----------- | ---------------------------------------------------------- |
| providers   | 由 Nest 注入器实例化的提供者，并且可以至少在整个模块中共享 |
| controllers | 必须创建的一组控制器                                       |
| imports     | 导入模块的列表，这些模块导出了此模块中所需提供者           |
| exports     | 由本模块提供并应在其他模块中可用的提供者的子集。           |

 事实上，根模块可能是应用程序中唯一的模块，特别是当应用程序很小时，但是对于大型程序来说，将所有功能都糅杂在一个模块中会变得难以维护，在大多数情况下，您将拥有多个模块，每个模块都有一组紧密相关的**功能**。 所以，更好的选择是将功能拆分到不同的模块中，然后导入到根模块中来。

使用`Nest-cli`创建模块也非常方便，使用`nest g module`命令即可快速地创建模块，比如`nest g module modules/api`，会在`modules`目录下创建`api`模块。

### 共享模块服务

模块之中的服务是可以共享的，假如有`share`模块，`user`模块想要使用`share`模块的服务，只需要在`share`模块的`@Module`中的`exports`中将共享模块暴露：

```typescript
@Module({
    providers: [ShareService],
    exports: [ShareService],	// 将共享的服务暴露
})
```

如果提供者是自定义的，那么应该这样导出，导出标识符：

```typescript
@Module({
    providers: [sequelizeProvider],
    exports: ['SEQUELIZE'], // 其他模块的组件直接使用@Inject('SEQUELIZE')即可
})
```

然后在`user`模块中的`@Module`中将`share`模块`imports`进来（注意，`exports`的是服务，`imports`的是模块）：

```typescript
@Module({
    imports: [ShareModule]	// 引入模块
    providers: [UserService], 
})
```

然后在`user`的控制器中的构造函数中定义`ShareService`即可获得注入的共享服务。

### 全局模块

上面定义的模块都是需要手动`imports`进来的，如果有些模块是使用率很高的，比如工具模块，此时可以声明为全局模块。

使用`@Global()`即可声明全局模块。

```typescript
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';

@Global()
@Module({
  controllers: [UserController],
  providers: [UserService],
})
export class catsModule {
  
}
```

 `@Global` 装饰器使模块成为全局作用域。 全局模块应该只注册一次，最好由根或核心模块注册。 在上面的例子中，`UserService` 组件将无处不在，而想要使用 `UserService` 的模块则不需要在 `imports` 数组中导入 `UserModule`。 

### 动态模块

上面定义的都是静态模块，如果我们需要动态声明我们的模块，比如数据库模块，连接成功才返回模块，此时需要使用动态模块来处理。

使用`模块名.forRoot()`方法来返回模块定义，通过该方式定义的即为动态模块。

```typescript
@Module({
    providers: [DatabaseProvider]
})
export class DatabaseModule {
    static async forRoot(env: string) {
         const provider =  createDatabaseProvider(env); // 根据环境变量连接不同的数据库
         return {
             module: DatabaseModule,
             providers: [provider],
             exports: [provider]
         }
    }
}

```

```typescript
// user.module.ts
@Module({
    imports: [DatabaseModule.forRoot('production')]
})
export class UserModule {}
```

如果要在全局范围内注册动态模块，请将 `global` 属性设置为 `true`。

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

## 中间件

 中间件是在路由处理程序 **之前** 调用的函数。 中间件函数可以访问请求和响应对象，以及应用程序请求响应周期中的 `next()` 中间件函数。 `next()` 中间件函数通常由名为 `next` 的变量表示。  中间件中如果想往下 匹配的话，那么需要写`next()`。

`Nest` 中间件实际上等价于 [express](http://expressjs.com/en/guide/using-middleware.html) 中间件。 下面是`Express`官方文档中所述的中间件功能：

中间件函数可以执行以下任务:

- 执行任何代码。
- 对请求和响应对象进行更改。
- 结束请求-响应周期。
- 调用堆栈中的下一个中间件函数。
- 如果当前的中间件函数没有结束请求-响应周期, 它必须调用 `next()` 将控制传递给下一个中间件函数。否则, 请求将被挂起。

### 类中间件

 `Nest`允许你使用函数或者类来实现自己的中间件。如果用类实现，则需要使用 `@Injectable()` 装饰，并且实现 `NestMiddleware` 接口。 如果使用`Nest-cli`可以使用命令`nest g middleware`快速生成类中间件，比如`nest g middleware middleware/LoggerMiddleware`，这会在`src/middleware`下生成中间件。

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

 中间件不能在 `@Module()` 装饰器中列出。必须使用模块类的 `configure()` 方法来设置它们。包含中间件的模块必须实现 `NestModule` 接口。我们将 `LoggerMiddleware` 设置在 `ApplicationModule` 层上。 

```typescript
@Module({
  imports: [CatsModule],
})
export class ApplicationModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)	// 应用中间件
      .forRoutes('cats');	// 指定中间件的作用路径
  }
}
```

上面的代码 `forRoutes` 方法表示只将中间件应用在 `cats` 路由上，还可以是指定的 `HTTP` 方法，甚至是路由通配符：

```typescript
.forRoutes({ path: 'cats', method: RequestMethod.GET });
.forRoutes({ path: 'ab*cd', method: RequestMethod.ALL });
.forRoutes({ path: '*', method: RequestMethod.ALL }); // 匹配所有路由
// forRoutes 可以写多个配置
.forRoutes({ path: 'a', method: RequestMethod.ALL }, { path: 'b', method: RequestMethod.GET });
```

当然，你也可以指定不包括某些路由规则：

```ts
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST }
  )
  .forRoutes(CatsController);
```

不过请注意 `exclude` 方法不能运用在函数式的中间件上，而且这里指定的 `path` 也不支持通配符，这只是个快捷方法，如果你真的需要某种路由级别的控制，那完全可以把逻辑写在一个单独的中间件中。

### 函数中间件

 一个没有额外的方法和依赖关系的中间件也可以转换为函数式中间件，这和`Express`中的中间件形式非常接近：

```typescript
export function logger(req, res, next) {
  console.log(`Request...`);
  // 如果要运行下个路由，一定要写 next()
  // 否则会一直挂在这里
  next();
};
```

### 多个中间件

`apply` 方法传入多个中间件参数即可：

```typescript
consumer.apply(cors(), helmet(), logger)
// forRoutes也可以直接传入控制器，意思是只有这个控制器作用中间件
.forRoutes(CatsController);

// logger作用于全部路径
// userMiddleware 作用于user路径
consumer.apply(logger)
.forRoutes('*')
.apply(UserMiddleware)
.forRoutes('user')
```

### 全局中间件

在实现了 `INestApplication` 接口的实例上调用 `use()` 方法即可：

```typescript
const app = await NestFactory.create(ApplicationModule);
app.use(logger);
await app.listen(3000);
```

需要注意的是，全局中间件不能直接使用类中间件，而是应该传入类中间件实例的`use`方法：

```typescript
app.use(new LoggerMiddleware().use);
```

## 管道

 管道是具有 `@Injectable()` 装饰器的类。管道应实现 `PipeTransform` 接口。 

管道有两个类型:

- **转换**：管道将输入数据转换为所需的数据输出
- **验证**：对输入数据进行验证，如果验证成功继续传递; 验证失败则抛出异常;

在这两种情况下, 管道 `参数(arguments)` 会由控制器的路由处理程序进行处理. `Nest` 会在调用路由处理方法之前插入一个管道，管道会先拦截方法的调用参数,进行转换或是验证处理，然后用转换好或是验证好的参数调用原方法。

使用`Nest-cli`创建管道可以使用`nest g pipe 管道名`，创建完成后的管道如下：

```typescript
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';

// 使用@Injectable装饰
@Injectable()
// 实现 PipeTransform接口
export class UserPipe implements PipeTransform {
  // 实现里面的 transform方法
  transform(value: any, metadata: ArgumentMetadata) {
      //里面可以修改传入的值以及验证转入值的合法性
    return value;
  }
}
```

每个管道必须提供 `transform()` 方法。 这个方法有两个参数：

- `value`
- `metadata`

`value` 是当前处理的参数，而 `metadata` 是其元数据。元数据对象包含一些属性：

```typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';
  metatype?: Type<unknown>;
  data?: string;
}
```

这里有一些属性描述参数：

| 参数     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| type     | 告诉我们该属性是一个 body `@Body()`，query `@Query()`，param `@Param()` 还是自定义参数 [在这里阅读更多](https://docs.nestjs.cn/customdecorators)。 |
| metatype | 属性的元类型，例如 `String`。 如果在函数签名中省略类型声明，或者使用原生 JavaScript，则为 `undefined`。 |
| data     | 传递给装饰器的字符串，例如 `@Body('string')`。 如果您将括号留空，则为 `undefined`。 |

此时就可以在控制器中使用管道：

```typescript
import {
  Controller,
  Get,
  Query,
  UsePipes,
} from '@nestjs/common';
import { UserService } from './user.service';
// 引入管道
import { UserPipe } from '../user.pipe';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  // 使用管道 传入管道的实例
  @UsePipes(new UserPipe())
  getUser(@Query() query) {
    console.log(query);
    return this.userService.getUser();
  }
}

```

假如管道修改如下：

```typescript
export class UserPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    value.id = '哈哈哈';
    return value;
  }
}
```

那么此时访问`http://localhost:3000/user?id=啦啦啦`，会输出`{ id: '哈哈哈' }`。

管道会对注入的`body`，`query`，`param`和自定义装饰器进行操作。

### 使用 joi配合管道进行数据验证

能修改数据，自然也可以验证数据。比如这里使用`joi`库进行数据验证，首先安装：

```bash
yarn add joi
yarn add @types/joi -D
```

然后修改管道：

```typescript
import {
  ArgumentMetadata,
  Injectable,
  PipeTransform,
  BadRequestException,
} from '@nestjs/common';
import { ObjectSchema } from 'joi';

@Injectable()
export class UserPipe implements PipeTransform {
  constructor(private readonly schema: ObjectSchema) {}
  transform(value: any, metadata: ArgumentMetadata) {
    // 只对query进行验证
    if (metadata.type === 'query') {
      // 如果验证出错就返回400
      const { error } = this.schema.validate(value);
      if (error) {
        throw new BadRequestException('Validation failed');
      }
    }
    return value;
  }
}
```

在控制器中使用：

```typescript
import * as joi from 'joi';

// 定义 joi schema
const userSchema = joi.object().keys({
  id: joi.number().integer().min(0).required(),
  name: joi.string().required(),
  age: joi.number().integer().min(6).max(66).required(),
});

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  // 将 schema传入
  @UsePipes(new UserPipe(userSchema))
  getUser(@Query() query, @Param() param) {
    console.log(123, param);
    return this.userService.getUser();
  }
}

```

### 使用class-validator验证

如果使用的是`typescript`，则可以使用另一种方式进行验证。 [class-validator](https://github.com/pleerock/class-validator) 这个库允许使用基于装饰器的验证。装饰器的功能非常强大，尤其是与 `Nest` 的 `Pipe` 功能相结合使用时，因为我们可以通过访问 `metatype` 信息做很多事情，在开始之前需要安装一些依赖：

```bash
yarn add class-validator class-transformer
```

`class-validator`用来验证数据，`class-transformer`用来将字面量对象转变为对应类的实例对象。

安装完成以后，就可以对验证类添加验证装饰器：

```typescript
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

然后创建一个验证管道：

```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

@Injectable()
// PipeTransform 泛型一为value类型，泛型二为transform的返回类型
export class ValidationPipe implements PipeTransform<any> {
  // transform可以是异步的
  async transform(value: any, { metatype }: ArgumentMetadata) {
    // 如果是普通js那么metatype为undefined
    // 或者验证类型是原生 JavaScript 的数据类型时（没有附加验证装饰器），跳过验证
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    // plainToClass方法转换JavaScript的参数为可验证的类型对象
    // 这里把value转成metatype类型的实例对象， 即createCatDto
    const object = plainToClass(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

 当验证逻辑仅涉及一个指定的参数时，参数范围的管道非常有用。

 ```typescript
 @Post()
// 仅用在body上
async create(@Body(new ValidationPipe()) createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
 ```

由于 `ValidationPipe` 被创建为尽可能通用，所以我们将把它设置为一个**全局作用域**的管道，用于整个应用程序中的每个路由处理器。

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 作用到全局
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

## 异常过滤器

`Nest.js` 内置的**异常层**负责处理整个应用程序中的所有抛出的异常。当捕获到未处理的异常时，最终用户将收到友好的响应。 该过滤器处理类型 `HttpException`（及其子类）的异常。每个发生的异常都由全局异常过滤器处理, 当这个异常**无法被识别**时 (既不是 `HttpException` 也不是继承的类 `HttpException` ) , 用户将收到以下 `JSON` 响应:

```json
{
    "statusCode": 500,
    "message": "Internal server error"
}
```

### 基础异常类

`Nest`提供了一个内置的 `HttpException` 类，它从 `@nestjs/common` 包中导入。对于典型的基于`HTTP` `REST/GraphQL` `API`的应用程序，最佳实践是在发生某些错误情况时发送标准`HTTP`响应对象。

在 `CatsController`，我们有一个 `findAll()` 方法（`GET` 路由）。假设此路由处理程序由于某种原因引发异常，我们将对其进行如下编码：

```typescript
@Get()
async findAll() {
  // HttpStatus是从 @nestjs/common 包导入的枚举
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

现在当客户端调用这个端点时，响应如下所示：

```json
{
    "statusCode": 403,
    "message": "Forbidden"
}
```

`HttpException` 构造函数有两个必要的参数来决定响应:

- `response` 参数定义 `JSON` 响应体。它可以是 `string` 或 `object`，如下所述。
- `status`参数定义`HTTP`[状态码](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)。

默认情况下，`JSON` 响应主体包含两个属性：

- `statusCode`：默认为 `status` 参数中提供的 `HTTP` 状态代码
- `message`:基于状态的 `HTTP` 错误的简短描述

如果只想定义 `JSON` 响应主体的消息部分，可以 `response`参数中提供一个 `string`。

要定义整个 `JSON` 响应主体，可以在`response` 参数中传递一个`object`。 `Nest`将序列化对象，并将其作为`JSON` 响应返回。

第二个构造函数参数`status`是有效的 `HTTP` 状态代码。 最佳实践是使用从`@nestjs/common`导入的 `HttpStatus`枚举。

这是一个覆盖整个响应正文的示例：

```typescript
@Get()
async findAll() {
  throw new HttpException({
    code: 4001,
    status: HttpStatus.FORBIDDEN,
    error: 'This is a custom message',
  }, HttpStatus.FORBIDDEN);
}
```

总之，`HttpException`的第一个参数可以传字符串也可以传递对象，传递字符串时传入的是返回的`message`，传递对象时传入的时返回的整个响应`json`。

### 自定义异常

自定义异常通常是对常用的异常进行一层封装。自定义异常需要继承`HttpException`，比如这是一个自定义的`UserException`：

```typescript
export class UserException extends HttpException {
  constructor(errcode: number, errmsg: string, statusCode: number) {
    super({ errcode, errmsg }, statusCode);
  }
}
```

业务层在使用该异常时直接使用以下代码即可，将原来传递对象的代码扁平化了：

```typescript
throw new UserException(4001, '没有权限', HttpStatus.FORBIDDEN);
```

### 自定义异常过滤器

虽然内置的异常过滤器可以自动处理很多情况，但是我们无法完全控制异常处理过程，如果我们需要记录日志的话，使用内置的异常过滤器就无法做到，这时候可以使用`@Catch`装饰器来自定义异常过滤器。

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter<HttpException> {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
        // 记录日志
    console.log('%s %s error: %s', request.method, request.url, exception.message);
    // 发送响应
    response
      .status(status)
      .json({
        statusCode: status,
        message: exception.message
        path: request.url,
      });
  }
}
```

#### ArgumentHost

上面`ExceptionFilter`中的`catch`方法的参数`ArgumentHost`是原始请求的包装器，由于`NestJs`支持`HTTP/GRPC/WebSocket`，这三种请求的原始请求对象是有差异的，为了异常过滤器能够统一处理这三种异常，`NestJs`做了包装。最终在使用时处理那种异常由开发者来决定。 在此代码示例中，我们使用它来获取对 `Request` 和 `Response` 对象的引用 

`ArgumentHost`接口定义如下：

```typescript
export interface ArgumentsHost {
  getArgs<T extends Array<any> = any[]>(): T;
  getArgByIndex<T = any>(index: number): T;
  switchToRpc(): RpcArgumentsHost;
  switchToHttp(): HttpArgumentsHost;
  switchToWs(): WsArgumentsHost;
}
```

如果需要处理的是`WebSocket`异常，就使用`host.switchToWs()`，其他异常以此类推。

#### 使用自定义异常过滤器

使用`@UseFilters`注册自定义异常过滤器。

异常过滤器有以下三种作用范围：

- 方法级别
- 控制器级别
- 全局级别

##### 方法级别

只会处理该方法上抛出的异常，其他方法抛出的异常不会处理。

```typescript
@Post('login')
@UseFilters(UserExceptionFilter)
login(@Body('username') username:string, password: string) {
  throw new UserException(4001, '没有权限');
}
```

##### 控制器级别

只会处理该控制器方法上抛出的异常，其他控制器抛出的异常不处理。

```typescript
@Controller('user')
@UseFilters(UserExceptionFilter)
export class UserController {
  
}
```

##### 全局级别

在应用入口注册，不会对`Websocket`或者混合应用（同时支持两种应用，如`HTTP/GRPC`或者`HTTP/WebSocket`）生效。一般`Web`开发中全局异常过滤器已经够用了。

在main.ts中注册全局异常过滤器

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new UserExceptionFilter());
  await app.listen(3000);
}
bootstrap();
```

##### 依赖注入

由于异常过滤器并不是任何模块上下文的一部分，所以`NestJs`无法对其进行依赖注入管理，如果有此种需求，比如在异常过滤器中注入`service`，需要定义服务提供者。服务提供者名称为`NestJs`规定的常量`APP_FILTER`：

```typescript
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: UserExceptionFilter,
    },
  ],
})
export class AppModule {}
```

#### 捕获多种或所有异常

上例中提到的自定义异常处理器只会捕获`HttpException`异常，如果有系统异常，会使用内置的异常处理器。通过传入异常类型给`@Catch`装饰器来捕获多种异常。如果不传任何异常类型的话，`NestJs`会捕获所有异常（也就是`Error`及其子类）。

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch() // 捕获所有异常
export class HttpExceptionFilter implements ExceptionFilter<Error> {
  catch(exception: Error, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
        // @todo 记录日志
    console.log('%s %s error: %s', request.method, request.url, exception.message);
    // 发送响应
    response
      .status(status)
      .json({
        statusCode: status,
          message: exception.message
        path: request.url,
      });
  }
}
```

## 守卫

 守卫是一个使用 `@Injectable()` 装饰器的类。 守卫应该实现 `CanActivate` 接口。 守卫有一个单独的责任。它们确定请求是否应该由路由处理程序处理。到目前为止，访问限制逻辑大多在中间件内。这样很好，因为诸如 `token` 验证或将 `request` 对象附加属性与特定路由没有强关联。但中间件是非常笨的。它不知道调用 `next()` 函数后会执行哪个处理程序。另一方面，守卫可以访问 `ExecutionContext` 对象，所以我们确切知道将要执行什么。 

说白了：在`Nestjs`中如果我们想做权限判断的话可以在守卫中完成，也可以在中间件中完成。 

 守卫在每个中间件之后执行，但在任何拦截器或管道之前执行。 

`Nest-cli`使用`nest g guard`命令来创建守卫。

### 授权守卫

正如前面提到的，授权是保护的一个很好的用例，因为只有当调用者(通常是经过身份验证的特定用户)具有足够的权限时，特定的路由才可用。我们现在要构建的 `AuthGuard` 假设用户是经过身份验证的(因此，请求头附加了一个`token`)。它将提取和验证`token`，并使用提取的信息来确定请求是否可以继续。

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    // 返回true通过返回false不通过
    return validateRequest(request);
  }
}
```

`validateRequest()` 函数中的逻辑可以根据需要变得简单或复杂。本例的主要目的是说明保护如何适应请求/响应周期。

每个守卫必须实现一个`canActivate()`函数。此函数应该返回一个布尔值，指示是否允许当前请求。它可以同步或异步地返回响应(通过 `Promise` 或 `Observable`)。`Nest`使用返回值来控制下一个行为:

- 如果返回 `true`, 将处理用户调用。
- 如果返回 `false`, 则 `Nest` 将忽略当前处理的请求。

### 执行上下文

`canActivate()` 函数接收单个参数 `ExecutionContext` 实例。`ExecutionContext` 继承自 `ArgumentsHost` 。`ArgumentsHost` 是传递给原始处理程序的参数的包装器，在上面的示例中，我们只是使用了在 `ArgumentsHost`上定义的帮助器方法来获得对请求对象的引用。

`ExecutionContext` 提供了更多功能，它扩展了 `ArgumentsHost`，但是也提供了有关当前执行过程的更多详细信息。

```typescript
export interface ExecutionContext extends ArgumentsHost {
  getClass<T = any>(): Type<T>;
  getHandler(): Function;
}
```

`getHandler()`方法返回对将要调用的处理程序的引用。`getClass()`方法返回这个特定处理程序所属的 `Controller` 类的类型。例如，如果当前处理的请求是 `POST` 请求，目标是 `CatsController`上的 `create()` 方法，那么 `getHandler()` 将返回对 `create()` 方法的引用，而 `getClass()`将返回一个`CatsControllertype`(而不是实例)。

### 使用守卫

与管道和异常过滤器一样，守卫可以是控制器范围的、方法范围的或全局范围的。下面，我们使用 `@UseGuards()`装饰器设置了一个控制器范围的守卫。这个装饰器可以使用单个参数，也可以使用逗号分隔的参数列表。也就是说，你可以传递几个守卫并用逗号分隔它们。

```typescript
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

`@UseGuards()` 装饰器需要从 `@nestjs/common` 包导入。

上例，我们已经传递了 `RolesGuard` 类型而不是实例, 让框架进行实例化，并启用了依赖项注入。与管道和异常过滤器一样，我们也可以传递一个实例:

```typescript
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

上面的构造将守卫附加到此控制器声明的每个处理程序。如果我们决定只限制其中一个, 我们只需要在方法级别设置守卫。

```typescript
@Get('info')
@UseGuards(UserGuard)
info() {
  return {username: 'user'};
}
```

为了绑定全局守卫, 我们使用 `Nest` 应用程序实例的 `useGlobalGuards()` 方法:

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

对于混合应用程序，`useGlobalGuards()` 方法不会为网关和微服务设置守卫。对于“标准”(非混合)微服务应用程序，`useGlobalGuards()`在全局安装守卫。

全局守卫用于整个应用程序, 每个控制器和每个路由处理程序。在依赖注入方面, 从任何模块外部注册的全局守卫 (如上面的示例中所示) 不能插入依赖项, 因为它们不属于任何模块。为了解决此问题, 您可以使用以下构造直接从任何模块设置一个守卫:

```typescript
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

当使用此方法为守卫程序执行依赖项注入时，请注意，无论使用此构造的模块是什么，守卫程序实际上是全局的。

### 反射

 反射主要是指程序可以访问、检测和修改它本身状态或行为的一种能力。 

通过反射我们可以将权限信息轻松地放到路由处理程序之上。

这里使用反射来进行基于角色的权限验证：

#### 定义角色装饰器

这个装饰器的作用在于给路由处理程序添加这个路由的角色元数据。 `Nest`提供了通过 `@SetMetadata()` 装饰器将定制元数据附加到路由处理程序的能力。 

```
// roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

#### 定义控制器

假设我们有一个只允许管理员访问的创建用户的接口：

```
@Post('create')
// 只传入admin这一个角色
@Roles('admin')
async create(@Body() createUserDTO: CreateUserDTO) {
  this.userService.create(createUserDTO);
}
```

#### 定义路由守卫

 为了访问路由的角色(自定义元数据)，我们将使用在 `@nestjs/core` 中提供的 `Reflector` 帮助类。 

```
// role.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  // 注入 reflector
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // 获取roles元数据
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) { // 未被装饰器装饰，直接放行
      return true;
    }
    const request = context.switchToHttp().getRequest();
    // 读取请求对象的user
    // 这个可以通过中间件拿到信息后挂载到request上
    const user = request.user; 
    const hasRole = () => user.roles.some((role) => roles.includes(role));
    return user && user.roles && hasRole();
  }
}
```

### 异常处理

路由守卫返回`false`时框架会抛出`ForbiddenException`，客户端收到的默认响应如下：

```
{
  "statusCode": 403,
  "message": "Forbidden resource"
}
```

如果需要抛出其他异常，比如`UnauthorizedException`，可以直接在路由守卫的`canActivate()`方法中抛出。

```typescript
throw new UnauthorizedException();
```

 由守卫引发的任何异常都将由异常层(全局异常过滤器和应用于当前上下文的任何异常过滤器)处理。 

## 拦截器

拦截器是一个实现了`NestInterceptor`接口且被`@Injectable`装饰器修饰的类。

拦截器具有一系列有用的功能，这些功能受面向`切面编程（AOP）`技术的启发。它们可以：

- 在函数执行之前/之后绑定**额外的逻辑**
- 转换从函数返回的结果
- **转换**从函数抛出的异常
- 扩展基本函数行为
- 根据所选条件完全重写函数 (例如, 缓存目的 ，一旦有可用的缓存则直接返回，不执行真正的业务逻辑，即业务逻辑处理函数行为已经被重写 )

 每个拦截器都有 `intercept()` 方法，它接收`2`个参数。

```typescript
function intercept(context: ExecutionContext, next: CallHandler): Observable<any>
```

 第一个是 `ExecutionContext` 实例（与守卫完全相同的对象）。 `ExecutionContext` 继承自 `ArgumentsHost` 。 `ArgumentsHost` 是传递给原始处理程序的参数的一个包装 ，它根据应用程序的类型包含不同的参数数组。 

第二个参数是 `CallHandler`。该接口是对路由处理函数的抽象，接口定义如下：

```typescript
export interface CallHandler<T = any> {
    handle(): Observable<T>;
}
```

如果不手动调用 `handle()` 方法，则路由处理程序根本不会执行。基本上，`CallHandler`是一个包装执行流的对象，因此推迟了最终的处理程序执行。

比方说，有人发送了 `POST /cats` 请求。此请求指向在 `CatsController` 中定义的 `create()` 处理程序。如果在此过程中未调用拦截器的 `handle()` 方法，则 `create()` 方法不会被执行。只有 `handle()` 被调用（并且已返回值），最终方法才会被触发。为什么？因为`Nest`订阅了返回的流，并使用此流生成的值来为最终用户创建单个响应或多个响应。而且，`handle()` 返回一个 `Observable`，一旦通过`Observable`接收到响应流，就可以对该流执行其他操作，并将最终结果返回给调用方。

`handle()`函数的返回值也就是对应路由函数的返回值。

以获取用户列表为例：

```typescript
// user.controller.ts
@Controller('user')
export class UserController {
  @Get()
  list() {
    return [];
  }
}
```

当访问 `/user/list` 时，路由处理函数返回`[]`，如果在应用拦截器的情况下，调用`CallHandler`接口的`handle()`方法得到的也是`Observable<[]>`(`RxJs`包装对象)。

**所以，如果在拦截器中调用了`next.handle()`方法就会执行对应的路由处理函数，如果不调用的话就不会执行。**

### 一个请求链路日志记录拦截器

随着微服务的兴起，原来的单一项目被拆分成多个比较小的子模块，这些子模块可以独立开发、独立部署、独立运行，大大提高了开发、执行效率，但是带来的问题也比较多，一个经常遇到的问题是接口调用出错不好查找日志。

如果在业务逻辑中硬编码这种链路调用日志是非常不可取的，严重违反了单一职责的原则，这在微服务开发中是相当不好的一种行为，会让微服务变得臃肿，这些逻辑完全可以通过拦截器来实现。

```typescript
// app.interceptor.ts
import { CallHandler, ExecutionContext, Injectable, Logger, NestInterceptor } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { Request } from 'express';
import { format } from 'util';

@Injectable()
export class AppInterceptor implements NestInterceptor {
  private readonly logger = new Logger(); // 实例化日志记录器

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now(); // 请求开始时间

    return next.handle().pipe(tap((response) => {
      // 调用完handle()后得到RxJs响应对象，使用tap可以得到路由函数的返回值
      const host = context.switchToHttp();
      const request = host.getRequest<Request>();

      // 打印请求方法，请求链接，处理时间和响应数据
      this.logger.log(format(
        '%s %s %dms %s',
        request.method,
        request.url,
        Date.now() - start,
        JSON.stringify(response),
      ));
    }));
  }
}
```

 `NestInterceptor` 是一个通用接口，其中 `T` 表示已处理的 `Observable` 的类型（在流后面），而 `R` 表示包含在返回的 `Observable` 中的值的返回类型。 

### 使用拦截器

拦截器可以在以下作用域进行绑定：

- 全局拦截器
- 控制器拦截器
- 路由方法拦截器

#### 全局拦截器

在main.ts中使用以下代码即可：

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new AppInterceptor());
```

#### 控制器拦截器

将对该控制器所有**路由**方法生效：

```typescript
@Controller('user')
@UseInterceptors(AppInterceptor)
export class UserController {
}
```

#### 方法拦截器

只对当前被装饰的路由方法进行拦截：

```typescript
@Controller('user')
export class UserController {
  @UseInterceptors(AppInterceptor)
  @Get()
  list() {
    return [];
  }
}
```

#### 依赖注入

在依赖注入方面, 从任何模块外部注册的全局拦截器 (如上面的示例中所示) 无法插入依赖项, 因为它们不属于任何模块。为了解决此问题, 您可以使用以下构造**直接从任何模块**设置一个拦截器:

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

### 响应处理

`CallHandler`接口的`handle()`返回值实际上是`RxJs`的`Observable`对象，利用`RxJs`操作符可以对该对象进行操作，比如有一个`API`接口，之前返回的数据结构如下，如果正常响应，响应体就是数据，没有包装结构：

```typescript
{
  "id":1,
  "name":"daming"
}
```

新的需求是要把之前的纯数据响应包装为一个`data`属性，结构如下：

```typescript
{
  "data": {
    "id": 1,
    "name":"daming"
  }
}
```

使用`NestJs`的拦截器，可以快速地实现该需求。

```typescript
import { CallHandler, ExecutionContext, Injectable, Logger, NestInterceptor } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class AppInterceptor implements NestInterceptor {

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {

    return next.handle().
      pipe(map(data => ({ data }))); // map操作符与Array.prototype.map类似
  }
}
```

应用上述拦截器后响应数据就会被包上一层`data`属性。

拦截器在创建用于整个应用程序的可重用解决方案时具有巨大的潜力。例如，我们假设我们需要将每个发生的 `null` 值转换为空字符串 `''`。我们可以使用一行代码并将拦截器绑定为全局代码。由于这一点，它会被每个注册的处理程序自动重用。

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class ExcludeNullInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(map(value => value === null ? '' : value ));
  }
}
```

### 异常处理

另外一个有趣的例子是利用`RxJs`的`catchError`来覆盖路由处理函数抛出的异常。

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  BadGatewayException,
  CallHandler,
} from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(
        catchError(err => throwError(new BadGatewayException())) // catchError用来捕获异常
      );
  }
}
```

## 重写路由函数逻辑

拦截器可以重写路由处理函数逻辑。如下是一个缓存拦截器的例子

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable, of } from 'rxjs';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(private readonly cacheService: CacheService) {}
  
   async intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
      const host = context.switchToHttp();
    const request = host.getRequest();
    if(request.method !== 'GET') {  
      // 非GET请求放行
      return next.handle();
    }
    const cachedData = await this.cacheService.get(request.url);
    if(cachedData) { 
      // 命中缓存，直接放行
      return of(cachedData);
    }
    return next.handle().pipe(tap(response) => { 
      // 响应数据写入缓存，此处可以等待缓存写入完成，也可以不等待
      this.cacheService.set(request.method, response);
    });
  }
}
```

由于`handle()`返回的是`rxjs`流，所以可以使用更多操作符进行操作。

# 技术

## cookie

`Nest.js`的`cookie`解析是基于平台的，比如`Express`平台可以使用`Express`的`cookie`中间件`cookie-parser`。

首先安装`cookie-parser`和类型文件`@types/cookie-parser`：

```bash
yarn add cookie-parser
yarn add cookie-parser -D
```

安装完成后，将`cookie-parser`配置为全局中间件（例如在`main.ts`文件中）。

```TypeScript
import * as cookieParser from 'cookie-parser';
app.use(cookieParser());
```

可以向`cookieParser`中间件中传递一些参数：

- `secret`： 一个字符串或者数组，用来给`cookie`签名。如果不指定这个选项，将不解析签名的`cookie`。如果提供了一个字符串，那么它会被用来作为`secret`。如果提供了一个数组，将尝试依次使用其元素来作为`secret`解析`cookie`。
- `option`：一个作为第二个参数传递给`cookie.parse`的对象，参见`[cookie](https://www.npmjs.org/package/cookie)`来了解更多内容。

该中间件将从请求的头文件中解析`Cookie`并将其数据作为`req.cookies`暴露出来。如果提供了`secret`，将暴露为`req.signedCokkies`。这些属性以`cookie`名称和属性的键值对保存。

当提供了`secret`时，该中间件将解析并验证所有签名的`cookie`并将其值从`req.cookies`移动到`req.signedCookies`。签名`cookie`是指包含`s:`前缀的`cookie`。验证失败的签名`cookie`值会被替换为`false`而不是被篡改过的值。

当这些完成后，就可以从路径处理程序中读取`cookie`了，例如：

```TypeScript
@Get()
findAll(@Req() request: Request) {
  console.log(request.cookies); // or "request.cookies['cookieKey']"
  // or console.log(request.signedCookies);
}
```

要在输出的响应中附加`cookie`，使用`Response.cookie()`方法：

```TypeScript
@Get()
findAll(@Res({ passthrough: true }) response: Response) {
  response.cookie('key', 'value', ,{maxAge: 900000, httpOnly: true});
}
```

第三个参数可以有以下属性：

| 属性     | 类型    | 说明                                                         |
| -------- | ------- | ------------------------------------------------------------ |
| domain   | String  | 域名                                                         |
| expires  | Date    | 过 期 时 间 （ 秒 ） ， 在 设 置 的 某 个 时 间 点 后 该 Cookie 就 会 失 效 ， 如 expires=Wednesday, 09-Nov-99 23:12:40 GMT |
| maxAge   | Number  | 最大失效时间（毫秒），设置在多少后失效                       |
| secure   | Boolean | 当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效 |
| path     | String  | 表示 cookie 影响到的路径，如 path=/。如果路径不能匹配时，浏览器则不发送这 个 Cookie |
| httpOnly | Boolean | 是微软对 COOKIE 做的扩展。如果在 COOKIE 中设置了“httpOnly”属性，则通 过程序（JS 脚本、applet 等）将无法读取到 COOKIE 信息，防止 XSS 攻击产生 |
| signed   | Boolean | 表 示 是 否 签 名 cookie, 设 为 true 会 对 这 个 cookie 签 名 ， 这 样 就 需 要 用 res.signedCookies 而不是 res.cookies 访问它。被篡改的签名 cookie 会被服务器拒绝，并且 cookie 值会重置为它的原始值 |

因为`cookie`是保存在浏览器端的，会存在一定的安全问题，如果想增加安全性，可以对`cookie`进行加密，设置加密后，服务器设置的`cookie`会进行加解密，以防止客户端的恶意篡改。

要设置`cookie`加密，需要在配置中间件的时候传参：

```typescript
app.use(coolieParser('secret'));
```

 要加密的 `cookie` 需要配置 `signed` 属性 ：

```typescript
res.cookie('secret-test', 'secret-value', { signed: true });
```

加密的`cookie`用`req.cookies`是获取不到的，需要使用` req.signedCookies `：

```typescript
console.log(req.signedCookies);
```

## session

 `session` 是另一种记录客户状态的机制，不同的是 `Cookie` 保存在客户端浏览器中，而 `session` 保存 在服务器上。 

 当浏览器访问服务器并发送第一次请求时，服务器端会创建一个 `session` 对象，生成一个类似于 `key,value` 的键值对，然后将 `key(cookie)`返回到浏览器端，浏览器下次再访问时，携带 `key(cookie)`， 找到对应的 `session(value)`。 客户的信息都保存在 `session` 中。

同样，`session`也是基于平台的，`Express`可以使用中间件`express-session`，首先安装：

```typescript
yarn add express-session
yarn add @types/express-session -D
```

然后在`main.ts`中作为全局中间件使用：

```typescript
import * as expressSession from 'express-session';
app.use(
    expressSession({
      secret: 'secret',
      cookie: { maxAge: 60000 },
      resave: false,
      saveUninitialized: true,
    }),
  );
```

这个中间件的常用参数有：

| 参数              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| secret            | 一个 String 类型的字符串，作为服务器端生成 session 的签名。  |
| name              | 返回客户端的 key 的名称，默认为 connect.sid,也可以自己设置。 |
| resave            | 强制保存 session 即使它并没有变化,。默认为 true。建议设置成 false。 |
| saveUninitialized | 强制将未初始化的 session 存储。当新建了一个 session 且未设定属性或值时，它就处于 未初始化状态。在设定一个 cookie 前，这对于登陆验证，减轻服务端存储压力，权限控制是有帮助的。（默 认：true）。建议手动添加。 |
| cookie            | 设置返回到前端 key 的属性，默认值为{ path: ‘/’, httpOnly: true, secure: false, maxAge: null }。 |
| rolling           | 在每次请求时强行设置 cookie，这将重置 cookie 过期时间（默认：false）。 |
|                   | 保存session的store，比如可以保存到redis中。                  |

常用方法：

```typescript
req.session.destroy(function(err) { // 销毁 session
})
req.session.username='张三'; //设置 session
req.session.username //获取 session
req.session.cookie.maxAge=0; //重新设置 cookie 的过期时间
```

每个`connect.sid`的`value`都作为`session`的`key`进行存储，对应的`value`可以理解为一个对象，对象中可以存储任何值，设置值时会设置这个对象的`key`和`value`。

如果要将`session`保存到`redis`中，可以这样配置：

```typescript
const redis = require('redis')
const session = require('express-session')
 
let RedisStore = require('connect-redis')(session)
let redisClient = redis.createClient()
 
app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: 'keyboard cat',
    resave: false,
  })
)
```

## 文件上传

 上传文件时 `From` 表单中需要配置 `enctype="multipart/form-data"`：

```html
<form action="user/add" method="post" enctype="multipart/form-data">
<input type="file" name="pic"/>
<input type="submit" value="提交">
</form>
```

### 单文件上传

 当我们要上传单个文件时, 我们只需将 `FileInterceptor()` 与处理程序绑定在一起, 然后使用 `@UploadedFile()` 装饰器从 `request` 中取出 `file`。 

```typescript
import {
  Controller,
  Post,
  UseInterceptors,
  UploadedFile,
  Body,
} from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';
import { createWriteStream } from 'fs';
import { join } from 'path';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post('doAdd')
  // 首先使用拦截器，FileInterceptor拦截文件
  // FileInterceptor 的参数为 上传文件的Input的name
  // 这里表示拦截 name 为 pic 的文件
  @UseInterceptors(FileInterceptor('pic'))
  // 然后使用@uploadedFile获取到拦截的文件的信息
  addUser(@UploadedFile() file, @Body() body) {
    console.log(body);
    console.log(file);
    // 使用node的createWriteStream 创建写入流
    const writeImage = createWriteStream(
      join(__dirname, '..', '../public/upload', `${file.originalname}`),
    );
    // 写入
    writeImage.write(file.buffer);
    return '上传成功';
  }
}
```

### 多文件上传

#### 文件数组

为了上传文件数组，我们使用 `FilesInterceptor()`。请使用 `FilesInterceptor()` 装饰器(注意装饰器名称中的复数)。这个装饰器有三个参数:

- `fieldName`:（保持不变）
- `maxCount`:可选的数字，定义要接受的最大文件数
- `options`:可选的 `MulterOptions` 对象 ， 这些 `MulterOptions` 等效于传入 `multer` 构造函数 ([此处](https://github.com/expressjs/multer#multeropts)有更多详细信息) 

使用 `FilesInterceptor()` 时，使用 `@UploadedFiles()` 装饰器从请求中提取文件。

```typescript
@Post('upload')
@UseInterceptors(FilesInterceptor('files'))
uploadFile(@UploadedFiles() files) {
  console.log(files);
}
```

#### 多个文件

要上传多个文件（全部使用不同的键），请使用 `FileFieldsInterceptor()` 装饰器。这个装饰器有两个参数:

- `uploadedFields`:对象数组，其中每个对象指定一个必需的 `name` 属性和一个指定字段名的字符串值(如上所述)，以及一个可选的 `maxCount` 属性(如上所述)
- `options`:可选的 `MulterOptions` 对象，如上所述

使用 `FileFieldsInterceptor()` 时，使用 `@UploadedFiles()` 装饰器从 `request` 中提取文件。

```typescript
@Post('upload')
@UseInterceptors(FileFieldsInterceptor([
  { name: 'avatar', maxCount: 1 },
  { name: 'background', maxCount: 1 },
]))
uploadFile(@UploadedFiles() files) {
  console.log(files);
}
```

#### 任何文件

要使用任意字段名称键上载所有字段，请使用 `AnyFilesInterceptor()` 装饰器。该装饰器可以接受如上所述的可选选项对象。

使用 `FileFieldsInterceptor()` 时，使用 `@UploadedFiles()` 装饰器从 `request` 中提取文件。

```typescript
@Post('upload')
@UseInterceptors(AnyFilesInterceptor())
uploadFile(@UploadedFiles() files) {
  console.log(files);
}
```

## 配置静态资源和模板

### 配置静态资源

在`Nest.js`中配置静态资源目录可以修改`main.ts`，如下：

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
// 因为 Nest 基于其他平台，要配置静态资源就要使用其他平台的能力
// 这里以常用的 Express 为例， 首先导入 NestExpressApplication
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';

async function bootstrap() {
  // 创建时，将NestExpressApplication以泛型传入Create方法
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  // 然后调用 useStaticAssets方法，参数为静态文件目录
  // 假如根目录存在 /public/images/test.png
  // 此时访问 localhost:3000/images/test.png即可得到这张图片
  app.useStaticAssets(join(__dirname, '..', 'public'));
  await app.listen(3000);
}
bootstrap();
```

如果要访问`localhost:3000/static/`才是静态资源目录，可以修改如下：

```typescript
app.useStaticAssets(join(__dirname, '..', 'public'), { prefix: '/static/' });
```

### 配置模板引擎

首先安装对应的模板引擎，比如`ejs`，`handleBars(hbs)`，这里以`hbs`为例：

```bash
yarn add hbs
```

然后修改`main.ts`：

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useStaticAssets(join(__dirname, '..', 'public'), { prefix: '/static/' });
  // 设置模板目录根路径
  app.setBaseViewsDir(join(__dirname, '..', 'views'));
  // 设置模板引擎
  app.setViewEngine('hbs');
  await app.listen(3000);
}
bootstrap();
```

因为上面已经定义了模板根路径为`views`，所以在根目录创建`views`文件夹，里面就可以直接写模板，也可以在里面再创建目录，比如这里创建`article/index.hbs`，内容如下：

```typescript
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8" />
  <title>Article</title>
</head>

<body>
  This is the Article page ! id is {{ id }} !
</body>

</html>
```

然后就可以在`controller`里返回模板需要的信息，修改`article.controller.ts`：

```typescript
import {
  Controller,
  Get,
  Param,
  Render,
} from '@nestjs/common';

@Controller('article')
export class ArticleController {
  @Get(':id')
  // 使用 @Render 定义模板路径
  // 这里写 article 和 article/index都是一样的
  // 这里只是做一个可以创建目录的示范，实际上不用创建article目录
  // 直接在views目录下创建article.hbs文件完全可以
  @Render('article/index')
  getArticle(@Param() params) {
    return { id: params.id };
  }
}
```

## 使用TypeORM操作MySQL

为了与 `SQL`和 `NoSQL` 数据库集成，`Nest` 提供了 `@nestjs/typeorm` 包。`Nest` 使用[TypeORM](https://github.com/typeorm/typeorm)是因为它是 `TypeScript` 中最成熟的对象关系映射器( `ORM` )。因为它是用 `TypeScript` 编写的，所以可以很好地与 `Nest` 框架集成。

### 连接数据库

为了开始使用它，我们首先安装所需的依赖项。

```bash
 yarn add  @nestjs/typeorm typeorm mysql
```

安装完成后就可以在根模块中引入：

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    // 引入TypeOrmModule
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      charset: 'utf8mb4',
      entities: ['./**/*.entity{.ts,.js}'],
      // synchronize: true 不应在生产模式使用，否则可能会丢失生产数据
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

`forRoot()` 方法支持所有`TypeORM`包中`createConnection()`函数暴露出的配置属性。其他一些额外的配置参数描述如下：

| 参数                | 说明                                                     |
| :------------------ | :------------------------------------------------------- |
| retryAttempts       | 重试连接数据库的次数（默认：10）                         |
| retryDelay          | 两次重试连接的间隔(ms)（默认：3000）                     |
| autoLoadEntities    | 如果为`true`,将自动加载实体(默认：false)                 |
| keepConnectionAlive | 如果未`true`，在应用程序关闭后连接不会关闭（默认：false) |

 更多连接选项见[这里](https://typeorm.io/#/connection-options) 。

另外，我们可以在根目录创建 `ormconfig.json` ，而不是将配置对象传递给 `forRoot()`。

```json
{
  "type": "mysql",
  "host": "localhost",
  "port": 3306,
  "username": "root",
  "password": "root",
  "database": "test",
  "entities": ["dist/**/*.entity{.ts,.js}"],
  "synchronize": true
}
```

然后，我们可以不带任何选项地调用 `forRoot()` :

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [TypeOrmModule.forRoot()],
})
export class AppModule {}
```

 一旦完成，`TypeORM` 的`Connection`和 `EntityManager` 对象就可以在整个项目中注入(不需要导入任何模块) ，例如：

```typescript
import { Connection } from 'typeorm';

@Module({
  imports: [TypeOrmModule.forRoot(), PhotoModule],
})
export class AppModule {
  constructor(private readonly connection: Connection) {}
}
```

### 创建实体（Entity）

 实体是一个映射到数据库表（或使用 `MongoDB` 时的集合）的类。 你可以通过定义一个新类来创建一个实体，并用`@Entity()`来标记，比如下面是一个文章实体类： 

```typescript
// src/entities/article.entity.ts

import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, DeleteDateColumn } from 'typeorm';


@Entity('article')
export class ArticleEntity {
  // PrimaryGeneratedColumn创建自增主键
  @PrimaryGeneratedColumn({
    type: 'int',
    comment: '主键id',
  })
  id: number;

  @Column('varchar', {
    nullable: false,
    comment: '文章标题',
  })
  title: string;

  @Column('varchar', {
    nullable: false,
    comment: '文章内容',
  })
  content: string;

  @Column({
    nullable: false,
    name: 'category_id',
    comment: '文章类别',
  })
  categoryID: string;

  @Column('varchar', {
    nullable: true,
    comment: '文章简介',
  })
  intro: string;

  @Column('varchar', {
    nullable: true,
    comment: '文章封面',
  })
  cover: string;

  @Column('varchar', {
    nullable: true,
    comment: '文章标签',
  })
  tags: string;

  @Column('enum', {
    nullable: false,
    default: 0,
    enum: [0, 1, 2],
    comment: '文章状态，0为编辑中，1为已发布，2为不可用',
  })
  status: number;

  @CreateDateColumn({
    type: 'timestamp',
    name: 'created_at',
    comment: '创建时间',
  })
  createdAt: Date;

  @UpdateDateColumn({
    type: 'timestamp',
    name: 'updated_at',
    comment: '最后更新时间',
  })
  updatedAt: Date;

  @DeleteDateColumn({
    type: 'timestamp',
    name: 'delete_at',
    comment: '删除',
  })
  deleteAt: Date;
}
```

假如要为`Article`实体使用替代表名，可以在`@Entity`中指定：`@Entity（“my_article”）`。 如果要为应用程序中的所有数据库表设置基本前缀，可以在连接选项中指定`entityPrefix`。

#### 实体列

##### 主列

每个实体必须至少有一个主列。 有几种类型的主列：

- `@PrimaryColumn()` 创建一个主列，它可以获取任何类型的任何值。你也可以指定列类型。 如果未指定列类型，则将从属性类型自动推断。

下面的示例将使用`int`类型创建 `id`，你必须在保存数据时手动分配。

```typescript
import { Entity, PrimaryColumn } from "typeorm";

@Entity()
export class User {
    @PrimaryColumn()
    id: number;
}
```

- `@PrimaryGeneratedColumn()` 创建一个主列，该值将使用自动增量值自动生成。 它将使用`auto-increment` /`serial` /`sequence`创建`int`列（取决于数据库）。 你不必在保存数据时手动分配其值，该值将会自动生成。

```typescript
import { Entity, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number;
}
```

- `@PrimaryGeneratedColumn("uuid")` 创建一个主列，该值将使用`uuid`自动生成。 `Uuid` 是一个独特的字符串 `id`。 你不必在保存之前手动分配其值，该值将自动生成。

```typescript
import { Entity, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class User {
    @PrimaryGeneratedColumn("uuid")
    id: string;
}
```

你也可以拥有复合主列：

```typescript
import { Entity, PrimaryColumn } from "typeorm";

@Entity()
export class User {
    @PrimaryColumn()
    firstName: string;

    @PrimaryColumn()
    lastName: string;
}
```

当您使用`save`保存实体时，它总是先尝试使用给定的实体 `ID（或 ids）`在数据库中查找实体。 如果找到 `id / ids`，则将更新数据库中的这一行。 如果没有包含 `id / ids` 的行，则会插入一个新行。

要通过 `id` 查找实体，可以使用`manager.findOne`或`repository.findOne`。 例：

```typescript
// 使用单个主键查找一个id
const person = await connection.manager.findOne(Person, 1);
const person = await connection.getRepository(Person).findOne(1);

// 使用复合主键找到一个id
const user = await connection.manager.findOne(User, { firstName: "Timber", lastName: "Saw" });
const user = await connection.getRepository(User).findOne({ firstName: "Timber", lastName: "Saw" });
```

##### 特殊列

有几种特殊的列类型可以使用：

- `@CreateDateColumn` 是一个特殊列，自动为实体插入日期。无需设置此列，该值将自动设置。
- `@UpdateDateColumn` 是一个特殊列，在每次调用实体管理器或存储库的`save`时，自动更新实体日期。无需设置此列，该值将自动设置。
- `@VersionColumn` 是一个特殊列，在每次调用实体管理器或存储库的`save`时自动增长实体版本（增量编号）。无需设置此列，该值将自动设置。

#### 列类型

`TypeORM` 支持所有最常用的数据库支持的列类型。 列类型是特定于数据库类型的，这为数据库架构提供了更大的灵活性。 你可以将列类型指定为`@ Column`的第一个参数 或者在`@Column`的列选项中指定，例如：

```typescript
@Column("int")
```

或

```typescript
@Column({ type: "int" })
```

如果要指定其他类型参数，可以通过列选项来执行。 例如:

```typescript
@Column("varchar", { length: 200 })
```

或

```typescript
@Column({ type: "int", length: 200 })
```

`mysql`支持以下类型：

 `int`, `tinyint`, `smallint`, `mediumint`, `bigint`, `float`, `double`, `dec`, `decimal`, `numeric`, `date`, `datetime`, `timestamp`, `time`, `year`, `char`, `varchar`, `nvarchar`, `text`, `tinytext`, `mediumtext`, `blob`, `longtext`, `tinyblob`, `mediumblob`, `longblob`, `enum`, `json`, `binary`, `geometry`, `point`, `linestring`, `polygon`, `multipoint`, `multilinestring`, `multipolygon`, `geometrycollection` 。

其他数据库类型参见[column-types](https://typeorm.io/#/entities/column-types-for-mysql--mariadb)。

##### enum列类型

`postgres`和`mysql`都支持`enum`列类型。 并有多种列定义方式：

使用typescript枚举：

```typescript
export enum UserRole {
    ADMIN = "admin",
    EDITOR = "editor",
    GHOST = "ghost"
}

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "enum",
        enum: UserRole,
        default: UserRole.GHOST
    })
    role: UserRole

}
```

> 注意：支持字符串，数字和异构枚举。

使用带枚举值的数组：

```typescript
export type UserRoleType = "admin" | "editor" | "ghost",

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "enum",
        enum: ["admin", "editor", "ghost"],
        default: "ghost"
    })
    role: UserRoleType
}
```

##### simple-json 列类型

有一种称为`simple-array`的特殊列类型，它可以将原始数组值存储在单个字符串列中。 所有值都以逗号分隔。 例如：

```typescript
@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column("simple-array")
    names: string[];
}
```

```typescript
const user = new User();
user.names = ["Alexander", "Alex", "Sasha", "Shurik"];
```

存储在单个数据库列中的`Alexander，Alex，Sasha，Shurik`值。 当你从数据库加载数据时，name 将作为 names 数组返回，就像之前存储它们一样。

注意**不能**在值里面有任何逗号。

##### 具有生成值的列

你可以使用`@Generated`装饰器创建具有生成值的列。 例如：

```typescript
@Entity()
export class User {
    @PrimaryColumn()
    id: number;

    @Column()
    @Generated("uuid")
    uuid: string;
}
```

`uuid`值将自动生成并存储到数据库中。

除了`"uuid"`之外，还有`"increment"`生成类型，但是对于这种类型的生成，某些数据库平台存在一些限制（例如，某些数据库只能有一个增量列，或者其中一些需要增量才能成为主键）。

#### 列选项

列选项定义实体列的其他选项。 你可以在`@Column`上指定列选项：

```typescript
@Column({
    type: "varchar",
    length: 150,
    unique: true,
    // ...
})
name: string;
```

`ColumnOptions`中可用选项列表：

- `type: ColumnType` - 列类型。
- `name: string` - 数据库表中的列名。默认情况下，列名称是从属性的名称生成的。 你也可以通过指定自己的名称来更改它。

- `length: number` - 列类型的长度。 例如，如果要创建`varchar（150）`类型，请指定列类型和长度选项。
- `width: number` - 列类型的显示范围。 仅用于[MySQL integer types(opens new window)](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)
- `onUpdate: string` - `ON UPDATE`触发器。 仅用于 [MySQL (opens new window)](https://dev.mysql.com/doc/refman/5.7/en/timestamp-initialization.html).
- `nullable: boolean` - 在数据库中使列`NULL`或`NOT NULL`。 默认情况下，列是`nullable：false`。
- `update: boolean` - 指示`"save"`操作是否更新列值。如果为`false`，则只能在第一次插入对象时编写该值。 默认值为`"true"`。
- `select: boolean` - 定义在进行查询时是否默认隐藏此列。 设置为`false`时，列数据不会显示标准查询。 默认情况下，列是`select：true`
- `default: string` - 添加数据库级列的`DEFAULT`值。
- `primary: boolean` - 将列标记为主要列。 使用方式和`@ PrimaryColumn`相同。
- `unique: boolean` - 将列标记为唯一列（创建唯一约束）。
- `comment: string` - 数据库列备注，并非所有数据库类型都支持。
- `precision: number` - 十进制（精确数字）列的精度（仅适用于十进制列），这是为值存储的最大位数。仅用于某些列类型。
- `scale: number` - 十进制（精确数字）列的比例（仅适用于十进制列），表示小数点右侧的位数，且不得大于精度。 仅用于某些列类型。
- `zerofill: boolean` - 将`ZEROFILL`属性设置为数字列。 仅在 `MySQL`中使用。 如果是`true`，`MySQL`会自动将`UNSIGNED`属性添加到此列。
- `unsigned: boolean` - 将`UNSIGNED`属性设置为数字列。 仅在 `MySQL` 中使用。
- `charset: string` - 定义列字符集。 并非所有数据库类型都支持。
- `collation: string` - 定义列排序规则。
- `enum: string[]|AnyEnum` - 在`enum`列类型中使用，以指定允许的枚举值列表。 你也可以指定数组或指定枚举类。
- `asExpression: string` - 生成的列表达式。 仅在[MySQL (opens new window)](https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html)中使用。
- `generatedType: "VIRTUAL"|"STORED"` - 生成的列类型。 仅在[MySQL (opens new window)](https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html)中使用。
- `hstoreType: "object"|"string"` -返回`HSTORE`列类型。 以字符串或对象的形式返回值。 仅在`Postgres`中使用。
- `array: boolean` - 用于可以是数组的 `postgres` 列类型（例如 `int []`）
- `transformer: { from(value: DatabaseType): EntityType, to(value: EntityType): DatabaseType }` - 用于将任意类型`EntityType`的属性编组为数据库支持的类型`DatabaseType`。

注意：大多数列选项都是特定于 `RDBMS` 的，并且在`MongoDB`中不可用。

#### 实体继承

你可以使用实体继承减少代码中的重复。

例如，你有`Photo`, `Question`, `Post` 三个实体:

```typescript
@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

    @Column()
    size: string;
}

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

    @Column()
    answersCount: number;
}

@Entity()
export class Post {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;

    @Column()
    viewCount: number;
}
```

正如你所看到的，所有这些实体都有共同的列：`id`，`title`，`description`。 为了减少重复并产生更好的抽象，我们可以为它们创建一个名为`Content`的基类：

```typescript
export abstract class Content {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    description: string;
}
@Entity()
export class Photo extends Content {
    @Column()
    size: string;
}

@Entity()
export class Question extends Content {
    @Column()
    answersCount: number;
}

@Entity()
export class Post extends Content {
    @Column()
    viewCount: number;
}
```

来自父实体的所有列（`relations`，`embeds` 等）（父级也可以扩展其他实体）将在最终实体中继承和创建。

#### 嵌入式实体

除了继承来减少重复，也可以使用`嵌入embedded columns`。

同样是创建基类，但是不使用继承，而是这样：

```typescript
import {Entity, Column} from "typeorm";

export class Name {
    
    @Column()
    first: string;
    
    @Column()
    last: string;
    
}

@Entity()
export class User {
    
    @PrimaryGeneratedColumn()
    id: string;
    
    // 这样嵌入
    @Column(type => Name)
    name: Name;
    
    @Column()
    isActive: boolean;
    
}
```

生成的表为：

```typescript
+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| nameFirst   | varchar(255) |                            |
| nameLast    | varchar(255) |                            |
| isActive    | boolean      |                            |
+-------------+--------------+----------------------------+
```



### 索引

#### 单列索引

你可以在要创建索引的列上使用`@Index`为特定列创建数据库索引。 也可以为实体的任何列创建索引。 例如：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Index()
  @Column()
  firstName: string;

  @Column()
  @Index()
  lastName: string;
}
```

还可以指定索引名称：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Index("name1-idx")
  @Column()
  firstName: string;

  @Column()
  @Index("name2-idx")
  lastName: string;
}
```

#### 唯一索引

要创建唯一索引，需要在索引选项中指定`{unique：true}`：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Index({ unique: true })
  @Column()
  firstName: string;

  @Column()
  @Index({ unique: true })
  lastName: string;
}
```

#### 联合索引

要创建具有多个列的索引，需要将`@Index`放在实体本身上，并指定应包含在索引中的所有列属性名称。 例如：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from "typeorm";

@Entity()
@Index(["firstName", "lastName"])
@Index(["firstName", "middleName", "lastName"], { unique: true })
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  middleName: string;

  @Column()
  lastName: string;
}
```

#### 空间索引

`MySQL` 和 `PostgreSQL`（当 `PostGIS` 可用时）都支持空间索引。

要在 MySQL 中的列上创建空间索引，请在使用空间类型的列（`geometry`，`point`，`linestring`，`polygon`，`multipoint`，`multilinestring`，`multipolygon`，`geometrycollection`）上添加`index`，其中`spatial：true`）：

```typescript
@Entity()
export class Thing {
  @Column("point")
  @Index({ spatial: true })
  point: string;
}
```

要在 `PostgreSQL` 中的列上创建空间索引，请在使用空间类型（`geometry`，`geography`）的列上添加带有`spatial：true`的`Index`：

```typescript
@Entity()
export class Thing {
  @Column("geometry", {
    spatialFeatureType: "Point",
    srid: 4326
  })
  @Index({ spatial: true })
  point: Geometry;
}
```

### 关系

关系是指两个或多个表之间的联系。关系基于每个表中的常规字段，通常包含主键和外键。

关系有三种：

| 名称          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| 一对一        | 主表中的每一行在外部表中有且仅有一个对应行。使用`@OneToOne()`装饰器来定义这种类型的关系 |
| 一对多/多对一 | 主表中的每一行在外部表中有一个或多的对应行。使用`@OneToMany()`和`@ManyToOne()`装饰器来定义这种类型的关系 |
| 多对多        | 主表中的每一行在外部表中有多个对应行，外部表中的每个记录在主表中也有多个行。使用`@ManyToMany()`装饰器来定义这种类型的关系 |



#### 关系选项

你可以为关系指定几个选项：

- `eager: boolean` - 如果设置为 `true`，则在此实体上使用`find *` 或`QueryBuilder`时，将始终使用主实体加载关系（当为`true`时，查询时不用指定关系表，将自动关联查询加载）， 只有在使用`find *`方法时才有效，并且只能用于关系的一方，在关系的两边使用`eager：true`是不允许的。 。
- `cascade: boolean` - 级联，如果设置为 `true`，则将插入相关对象并在数据库中更新，修改一方关系的另一方也会更新。
- `onDelete: "RESTRICT"|"CASCADE"|"SET NULL"` - 指定删除引用对象时外键的行为方式。
- `primary: boolean` - 指示此关系的列是否为主列。
- `nullable: boolean` -指示此关系的列是否可为空。 默认情况下是可空。



#### @JoinColumn装饰器

`@JoinColumn`不仅定义了关系的哪一侧包含带有外键的连接列，还允许自定义连接列名和引用的列名。

当我们设置`@JoinColumn`时，它会自动在数据库中创建一个名为`propertyName + referencedColumnName`的列。 例如：

```typescript
@ManyToOne(type => Category)
@JoinColumn() // 这个装饰器对于@ManyToOne是可选的，但@OneToOne是必需的
category: Category;
```

此代码将在数据库中创建`categoryId`列。 如果要在数据库中更改此名称，可以指定自定义连接列名称：

```typescript
@ManyToOne(type => Category)
@JoinColumn({ name: "cat_id" })
category: Category;
```

`Join列`始终是对其他一些列的引用（使用外键）。 默认情况下，关系始终引用相关实体的主列。 如果要与相关实体的其他列创建关系 - 你也可以在`@JoinColumn`中指定它们：

```typescript
@ManyToOne(type => Category)
@JoinColumn({ referencedColumnName: "name" })
category: Category;
```

该关系现在引用`Category`实体的`name`，而不是`id`。 该关系的列名将变为`categoryName`。

#### @JoinTable装饰器

`@ JoinTable`用于“多对多”关系。 联结表是由 `TypeORM` 自动创建的一个特殊的单独表，其中的列引用相关实体。 你可以使用`@JoinColumn`更改联结表及其引用列中的列名，你还可以更改生成的联结表的名称。

```typescript
@ManyToMany(type => Category)
@JoinTable({
    name: "question_categories" // 此关系的联结表的表名
    joinColumn: {
        name: "question",
        referencedColumnName: "id"
    },
    inverseJoinColumn: {
        name: "category",
        referencedColumnName: "id"
    }
})
categories: Category[];
```

如果目标表具有复合主键， 则必须将一组属性发送到`@JoinTable`。

#### 一对一

我们以`User`和`Profile`实体为例。

用户只能拥有一个配置文件，并且一个配置文件仅由一个用户拥有。

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  gender: string;

  @Column()
  photo: string;
}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";
import { Profile } from "./Profile";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // User 的 profile 属性 对应一个 Profile 实体
  @OneToOne(() => Profile)
  // @JoinColumn 表明这是主表，会在此表生成关联外键
  @JoinColumn()
  profile: Profile;
}
```

这里我们将`@OneToOne`添加到`profile`并将目标关系类型指定为`Profile`。 我们还添加了`@JoinColumn`，这是必选项并且只能在关系的一侧设置。 你设置`@JoinColumn`在哪个表，哪一方的表将包含一个`"relation id"`和目标实体表的外键。

此示例将生成以下表：

```bash
+-------------+--------------+----------------------------+
|                        profile                          |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| gender      | varchar(255) |                            |
| photo       | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
| profileId   | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

同样，`@JoinColumn`必须仅设置在关系的一侧且必须在数据库表中具有外键的一侧。

该例子展示如何保存这样的关系：

```typescript
const profile = new Profile();
profile.gender = "male";
profile.photo = "me.jpg";
// 拿到manager后使用save保存
await connection.manager.save(profile);

const user = new User();
user.name = "Joe Smith";
user.profile = profile;
// 两个表分别保存
await connection.manager.save(user);
```

启用级联后（即主表对应属性的配置中设置了` cascade : true`，比如`@OneToOne(() => Profile, profile => profile.user, { cascade: true })`），只需一次`save`调用即可保存此关系。

要拿到嵌套`profile`的`user`，必须在`FindOptions`中指定关系：

```typescript
const userRepository = connection.getRepository(User);
// 指定需要查询关系profile
const users = await userRepository.find({ relations: ["profile"] });
```

或者使用`QueryBuilder`:

```typescript
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.profile", "profile")
  .getMany();
```

通过在关系上启用预先加载，你不必指定关系或手动加入，它将始终自动加载。

关系可以是单向的和双向的。 单向是仅在一侧与关系装饰器的关系。 双向是与关系两侧的装饰者的关系。

我们刚刚创建了一个单向关系。 让我们将它改为双向：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  gender: string;

  @Column()
  photo: string;

  @OneToOne(() => User, user => user.profile) // 将另一面指定为第二个参数，指定反向关系的字段名
  user: User;
}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";
import { Profile } from "./Profile";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Profile, profile => profile.user) // 指定另一面作为第二个参数
  @JoinColumn()
  profile: Profile;
}
```

我们只是创建了双向关系。 注意，反向关系没有`@JoinColumn`。 `@JoinColumn`必须只在关系的一侧且拥有外键的表上。

双向关系允许你使用`QueryBuilder`从双方加入关系：

```typescript
const profiles = await connection
  .getRepository(Profile)
  .createQueryBuilder("profile")
  .leftJoinAndSelect("profile.user", "user")
  .getMany();
```

#### 多对一/一对多

以`User` 和 `Photo` 实体为例。 `User` 可以拥有多张 `photos`，但每张 `photo` 仅由一位 `user` 拥有。

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  // 在多的一方使用 ManyToOne，并且指定反向关系字段名
  @ManyToOne(() => User, user => user.photos)
  user: User;
}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
  // 在一的一方使用 OneToMany，自己反向作为photo的user
  @OneToMany(() => Photo, photo => photo.user)
  photos: Photo[];
}
```

这里我们将`@OneToMany`添加到`photos`属性中，并将目标关系类型指定为`Photo`。 你可以在`@ManyToOne` / `@OneToMany`关系中省略`@JoinColumn`（因为一对多外键一定在多的一方），除非你需要自定义关联列在数据库中的名称。 `@ManyToOne`可以单独使用，但`@OneToMany`必须搭配`@ManyToOne`使用。 如果你想使用`@OneToMany`，则需要`@ManyToOne`。 在你设置`@ManyToOne`的地方，相关实体将有`关联id`和外键。

此示例将生成以下表：

```bash
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| url         | varchar(255) |                            |
| userId      | int(11)      |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+
```

如何保存这种关系：

```typescript
const photo1 = new Photo();
photo1.url = "me.jpg";
await connection.manager.save(photo1);

const photo2 = new Photo();
photo2.url = "me-and-bears.jpg";
await connection.manager.save(photo2);

const user = new User();
user.name = "John";
user.photos = [photo1, photo2];
await connection.manager.save(user);
```

或者你可以选择：

```typescript
const user = new User();
user.name = "Leo";
await connection.manager.save(user);

const photo1 = new Photo();
photo1.url = "me.jpg";
photo1.user = user;
await connection.manager.save(photo1);

const photo2 = new Photo();
photo2.url = "me-and-bears.jpg";
photo2.user = user;
await connection.manager.save(photo2);
```

启用级联后，只需一次`save`调用即可保存此关系。

要在内部加载带有 `photos` 的 `user`，必须在`FindOptions`中指定关系：

```typescript
const userRepository = connection.getRepository(User);
const users = await userRepository.find({ relations: ["photos"] });

// 或者带有 user 的 photos

const photoRepository = connection.getRepository(Photo);
const photos = await photoRepository.find({ relations: ["user"] });
```

或者使用`QueryBuilder`:

```typescript
const users = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .getMany();

// or from inverse side

const photos = await connection
  .getRepository(Photo)
  .createQueryBuilder("photo")
  .leftJoinAndSelect("photo.user", "user")
  .getMany();
```

通过在关系上启用预先加载，你不必指定关系或手动加入,它将始终自动加载。

#### 多对多

以`Question` 和 `Category` 实体为例。 `Question` 可以有多个 `categories`, 每个 `category` 可以有多个 `questions`。

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";
import { Category } from "./Category";

@Entity()
export class Question {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  text: string;

  // 使用 ManyToMany 和 JoinTable
  @ManyToMany(() => Category)
  @JoinTable()
  categories: Category[];
}
```

`@JoinTable()`是`@ManyToMany`关系所必需的。 你必须把`@JoinTable`放在关系的一边。

此示例将生成以下表：

```bash
+-------------+--------------+----------------------------+
|                        category                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                        question                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| title       | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|              question_categories_category               |
+-------------+--------------+----------------------------+
| questionId  | int(11)      | PRIMARY KEY FOREIGN KEY    |
| categoryId  | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```

如何保存这种关系：

```typescript
const category1 = new Category();
category1.name = "animals";
await connection.manager.save(category1);

const category2 = new Category();
category2.name = "zoo";
await connection.manager.save(category2);

const question = new Question();
question.categories = [category1, category2];
await connection.manager.save(question);
```

启用级联后，只需一次`save`调用即可保存此关系。

要在 `categories` 里面加载 `question`，你必须在`FindOptions`中指定关系：

```typescript
const questionRepository = connection.getRepository(Question);
const questions = await questionRepository.find({ relations: ["categories"] });
```

或者使用`QueryBuilder`

```typescript
const questions = await connection
  .getRepository(Question)
  .createQueryBuilder("question")
  .leftJoinAndSelect("question.categories", "category")
  .getMany();
```

通过在关系上启用预先加载，你不必指定关系或手动加入，它将始终自动加载。

关系可以是单向的和双向的。 单向是仅在一侧与关系装饰器的关系。 双向是与关系两侧的装饰者的关系。

我们刚刚创建了一个单向关系。 让我们改为双向：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm";
import { Question } from "./Question";

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Question, question => question.categories)
  questions: Question[];
}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";
import { Category } from "./Category";

@Entity()
export class Question {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  text: string;

  @ManyToMany(() => Category, category => category.questions)
  @JoinTable()
  categories: Category[];
}
```

我们只是创建了双向关系。 注意，反向关系没有`@JoinTable`。 `@JoinTable`必须只在关系的一边。

双向关系允许您使用`QueryBuilder`从双方加入关系：

```typescript
const categoriesWithQuestions = await connection
  .getRepository(Category)
  .createQueryBuilder("category")
  .leftJoinAndSelect("category.questions", "question")
  .getMany();
```

### Entity Manager

使用`EntityManager`，你可以管理任何实体。 `EntityManager` 就像放一个实体存储库的集合的地方。

你可以通过`getManager（）`或`Connection`访问实体管理器。

```typescript
import { getManager } from "typeorm";
import { User } from "./entity/User";

const entityManager = getManager(); // 你也可以通过 getConnection().manager 获取
const user = await entityManager.findOne(User, 1);
user.name = "Umed";
await entityManager.save(user);
```

#### API

- `connection` - 使用`EntityManager`连接。

```typescript
const connection = manager.connection;
```

- `queryRunner` - `EntityManager`使用的查询运行器。仅在 EntityManager 的事务实例中使用。

```typescript
const queryRunner = manager.queryRunner;
```

- `transaction` - 提供在单个数据库事务中执行多个数据库请求的事务。

```typescript
await manager.transaction(async manager => {
  // NOTE: you must perform all database operations using the given manager instance
  // it's a special instance of EntityManager working with this transaction
  // and don't forget to await things here
  // 注意：你必须使用给定的管理器实例执行所有数据库操作，
  // 它是一个使用此事务的EntityManager的特殊实例。
  // 在这里处理一些操作
});
```

- `query` - 执行原始 `SQL` 查询。

```typescript
const rawData = await manager.query(`SELECT * FROM USERS`);
```

- `createQueryBuilder` - 创建用于构建 `SQL` 查询的 `query builder`。 

```typescript
const users = await manager
  .createQueryBuilder()
  .select()
  .from(User, "user")
  .where("user.name = :name", { name: "John" })
  .getMany();
```

- `hasId` - 检查给定实体是否已定义主列属性。

```typescript
if (manager.hasId(user)) {
  // ... 做一些需要的操作
}
```

- `getId` - 获取给定实体的主列属性值。 如果实体具有复合主键，则返回的值将是具有主列的名称和值的对象。

```typescript
const userId = manager.getId(user); // userId === 1
```

- `create` - 创建`User`的新实例。 接受具有用户属性的对象文字，该用户属性将写入新创建的用户对象。（可选）

```typescript
const user = manager.create(User); // same as const user = new User();
const user = manager.create(User, {
  id: 1,
  firstName: "Timber",
  lastName: "Saw"
}); // 和 const user = new User(); user.firstName = "Timber"; user.lastName = "Saw"; 一样
```

- `merge` - 将多个实体合并为一个实体。

```typescript
const user = new User();
manager.merge(User, user, { firstName: "Timber" }, { lastName: "Saw" }); // 和user.firstName = "Timber"; user.lastName = "Saw";一样
```

- `preload` - 从给定的普通 `javascript` 对象创建一个新实体。 如果实体已存在于数据库中，则它将加载它（以及与之相关的所有内容），将所有值替换为给定对象中的新值，并返回新实体。 新的实体实际上是从与新对象代替所有属性的数据库实体加载。

```typescript
const partialUser = {
  id: 1,
  firstName: "Rizzrak",
  profile: {
    id: 1
  }
};
const user = await manager.preload(User, partialUser);
// user将包含partialUser中具有partialUser属性值的所有缺失数据：
// { id: 1, firstName: "Rizzrak", lastName: "Saw", profile: { id: 1, ... } }
```

- `save` - 保存给定实体或实体数组。 如果实体已存在于数据库中，则会更新。 如果该实体尚未存在于数据库中，则将其插入。 它将所有给定实体保存在单个事务中（在实体管理器而不是事务性的情况下）。 还支持部分更新，因为跳过了所有未定义的属性。 为了使值为`NULL`，你必须手动将该属性设置为等于`null`。

```typescript
await manager.save(user);
await manager.save([category1, category2, category3]);
```

- `remove` - 删除给定的实体或实体数组。 它删除单个事务中的所有给定实体（在实体的情况下，管理器不是事务性的）。

```typescript
await manager.remove(user);
await manager.remove([category1, category2, category3]);
```

- `insert` - 插入新实体或实体数组。

```typescript
await manager.insert(User, {
  firstName: "Timber",
  lastName: "Timber"
});

await manager.insert(User, [
  {
    firstName: "Foo",
    lastName: "Bar"
  },
  {
    firstName: "Rizz",
    lastName: "Rak"
  }
]);
```

- `update` - 通过给定的更新选项或实体 `ID` 部分更新实体。

```typescript
await manager.update(User, { firstName: "Timber" }, { firstName: "Rizzrak" });
// 执行 UPDATE user SET firstName = Rizzrak WHERE firstName = Timber

await manager.update(User, 1, { firstName: "Rizzrak" });
// 执行 UPDATE user SET firstName = Rizzrak WHERE id = 1
```

- `delete` - 根据实体 `id` 或 `ids` 或其他给定条件删除实体：

```typescript
await manager.delete(User, 1);
await manager.delete(User, [1, 2, 3]);
await manager.delete(User, { firstName: "Timber" });
```

- `count` - 符合指定条件的实体数量。对分页很有用。

```typescript
const count = await manager.count(User, { firstName: "Timber" });
```

- `increment` - 增加符合条件的实体某些列值。

```typescript
await manager.increment(User, { firstName: "Timber" }, "age", 3);
```

- `decrement` - 减少符合条件的实体某些列值。

```typescript
await manager.decrement(User, { firstName: "Timber" }, "age", 3);
```

- `find` - 查找指定条件的实体。

```typescript
const timbers = await manager.find(User, { firstName: "Timber" });
```

- `findAndCount` - 查找指定条件的实体。 还会计算与给定条件匹配的所有实体数量，但忽略分页设置（`from`和`take` 选项）。

```typescript
const [timbers, timbersCount] = await manager.findAndCount(User, { firstName: "Timber" });
```

- `findByIds` - 按 ID 查找多个实体。

```typescript
const users = await manager.findByIds(User, [1, 2, 3]);
```

- `findOne` - 查找匹配某些 ID 或查找选项的第一个实体。

```typescript
const user = await manager.findOne(User, 1);
const timber = await manager.findOne(User, { firstName: "Timber" });
```

- `findOneOrFail` - 查找匹配某些 ID 或查找选项的第一个实体。 如果没有匹配，则 Rejects 一个 promise。

```typescript
const user = await manager.findOneOrFail(User, 1);
const timber = await manager.findOneOrFail(User, { firstName: "Timber" });
```

- `clear` - 清除给定表中的所有数据(`truncates/drops`)。

```typescript
await manager.clear(User);
```

- `getRepository` - 获取`Repository`以对特定实体执行操作。 

```typescript
const userRepository = manager.getRepository(User);
```

- `getTreeRepository` - 获取`TreeRepository`以对特定实体执行操作。

```typescript
const categoryRepository = manager.getTreeRepository(Category);
```

- `getMongoRepository` - 获取`MongoRepository`以对特定实体执行操作。

```typescript
const userRepository = manager.getMongoRepository(User);
```

- `getCustomRepository` - 获取自定义实体库。 

```typescript
const myUserRepository = manager.getCustomRepository(UserRepository);
```

- `release` - 释放实体管理器的查询运行器。仅在手动创建和管理查询运行器时使用。

```typescript
await manager.release();
```

### Repository

`Repository`就像`EntityManager`一样，但其操作仅限于具体实体。

你可以通过`getRepository（Entity）`，`Connection.getRepository`或`EntityManager.getRepository`访问存储库。

例如：

```typescript
import { getRepository } from "typeorm";
import { User } from "./entity/User";

const userRepository = getRepository(User); // 你也可以通过getConnection().getRepository()或getManager().getRepository() 获取
const user = await userRepository.findOne(1);
user.name = "Umed";
await userRepository.save(user);
```

有三种类型的存储库：

- `Repository` - 任何实体的常规存储库。
- `TreeRepository` - 用于树实体的`Repository`的扩展存储库（比如标有`@ Tree`装饰器的实体）。有特殊的方法来处理树结构。
- `MongoRepository` - 具有特殊功能的存储库，仅用于 `MongoDB`。

#### 在Nest中获取Repository

首先在全局插入实体，让`TypeORM`知道它的存在：

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Photo } from './photo/photo.entity';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      // 在全局的entities中导入
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

然后在需要的模块中使用`forFeature`：

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User } from './user.entity';

@Module({
  // 在当前模块中注册User存储库
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

 此模块使用 `forFeature()` 方法定义在当前范围中注册哪些存储库。这样，我们就可以使用 `@InjectRepository()`装饰器将 `UsersRepository` 注入到 `UsersService` 中 ：

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    // 注入User
    @InjectRepository(User)
    private usersRepository: Repository<User>
  ) {}

  findAll(): Promise<User[]> {
    // 此时就可以使用userRespository
    return this.usersRepository.find();
  }

  findOne(id: string): Promise<User> {
    return this.usersRepository.findOne(id);
  }

  async remove(id: string): Promise<void> {
    await this.usersRepository.delete(id);
  }
}
```



#### API

- `manager` - 存储库使用的`EntityManager`。

```typescript
const manager = repository.manager;
```

- `metadata` - 存储库管理的实体的`EntityMetadata`。 

```typescript
const metadata = repository.metadata;
```

- `queryRunner` - `EntityManager`使用的查询器。仅在 `EntityManager` 的事务实例中使用。

```typescript
const queryRunner = repository.queryRunner;
```

- `target` - 此存储库管理的目标实体类。仅在 `EntityManager `的事务实例中使用。

```typescript
const target = repository.target;
```

- `createQueryBuilder` - 创建用于构建 `SQL` 查询的查询构建器。 

```typescript
const users = await repository
  .createQueryBuilder("user")
  .where("user.name = :name", { name: "John" })
  .getMany();
```

- `hasId` - 检查是否定义了给定实体的主列属性。

```typescript
if (repository.hasId(user)) {
  // ... do something
}
```

- `getId` - 获取给定实体的主列属性值。复合主键返回的值将是一个具有主列名称和值的对象。

```typescript
const userId = repository.getId(user); // userId === 1
```

- `create` - 创建`User`的新实例。 接受具有用户属性的对象文字，该用户属性将写入新创建的用户对象（可选）。

```typescript
const user = repository.create(); // 和 const user = new User();一样
const user = repository.create({
  id: 1,
  firstName: "Timber",
  lastName: "Saw"
}); // 和const user = new User(); user.firstName = "Timber"; user.lastName = "Saw";一样
```

- `merge` - 将多个实体合并为一个实体。

```typescript
const user = new User();
repository.merge(user, { firstName: "Timber" }, { lastName: "Saw" }); // 和 user.firstName = "Timber"; user.lastName = "Saw";一样
```

- `preload` - 从给定的普通 `javascript` 对象创建一个新实体。 如果实体已存在于数据库中，则它将加载它（以及与之相关的所有内容），并将所有值替换为给定对象中的新值，并返回新实体。 新实体实际上是从数据库加载的所有属性都替换为新对象的实体。

```typescript
const partialUser = {
  id: 1,
  firstName: "Rizzrak",
  profile: {
    id: 1
  }
};
const user = await repository.preload(partialUser);
// user将包含partialUser中具有partialUser属性值的所有缺失数据：
// { id: 1, firstName: "Rizzrak", lastName: "Saw", profile: { id: 1, ... } }
```

- `save` - 保存给定实体或实体数组。   如果该实体已存在于数据库中，则会更新该实体。   如果数据库中不存在该实体，则会插入该实体。   它将所有给定实体保存在单个事务中（在实体的情况下，管理器不是事务性的）。   因为跳过了所有未定义的属性，还支持部分更新。

```typescript
await repository.save(user);
await repository.save([category1, category2, category3]);
```

- `remove` - 删除给定的实体或实体数组。
- 它将删除单个事务中的所有给定实体（在实体的情况下，管理器不是事务性的）。

```typescript
await repository.remove(user);
await repository.remove([category1, category2, category3]);
```

- `insert` - 插入新实体或实体数组。

```typescript
await repository.insert({
  firstName: "Timber",
  lastName: "Timber"
});

await manager.insert(User, [
  {
    firstName: "Foo",
    lastName: "Bar"
  },
  {
    firstName: "Rizz",
    lastName: "Rak"
  }
]);
```

- `update` - 通过给定的更新选项或实体 `ID` 部分更新实体。

```typescript
await repository.update({ firstName: "Timber" }, { firstName: "Rizzrak" });
// 执行 UPDATE user SET firstName = Rizzrak WHERE firstName = Timber

await repository.update(1, { firstName: "Rizzrak" });
// 执行 UPDATE user SET firstName = Rizzrak WHERE id = 1
```

- `delete` -根据实体 `id`, `ids` 或给定的条件删除实体：

```typescript
await repository.delete(1);
await repository.delete([1, 2, 3]);
await repository.delete({ firstName: "Timber" });
```

- `count` - 符合指定条件的实体数量。对分页很有用。

```typescript
const count = await repository.count({ firstName: "Timber" });
```

- `increment` - 增加符合条件的实体某些列值。

```typescript
await manager.increment(User, { firstName: "Timber" }, "age", 3);
```

- `decrement` - 减少符合条件的实体某些列值。

```typescript
await manager.decrement(User, { firstName: "Timber" }, "age", 3);
```

- `find` - 查找指定条件的实体。

```typescript
const timbers = await repository.find({ firstName: "Timber" });
```

- `findAndCount` - 查找指定条件的实体。还会计算与给定条件匹配的所有实体数量， 但是忽略分页设置 (`skip` 和 `take` 选项)。

```typescript
const [timbers, timbersCount] = await repository.findAndCount({ firstName: "Timber" });
```

- `findByIds` - 按 `ID` 查找多个实体。

```typescript
const users = await repository.findByIds([1, 2, 3]);
```

- `findOne` - 查找匹配某些 `ID` 或查找选项的第一个实体。

```typescript
const user = await repository.findOne(1);
const timber = await repository.findOne({ firstName: "Timber" });
```

- `findOneOrFail` - - `findOneOrFail` - 查找匹配某些 `ID` 或查找选项的第一个实体。 如果没有匹配，则 `Rejects` 一个 `promise`。

```typescript
const user = await repository.findOneOrFail(1);
const timber = await repository.findOneOrFail({ firstName: "Timber" });
```

- `query` - 执行原始 `SQL`查询。

```typescript
const rawData = await repository.query(`SELECT * FROM USERS`);
```

- `clear` - 清除给定表中的所有数据(`truncates/drops`)。

```typescript
await repository.clear();
```

#### 其他选项

`SaveOptions`选项可以传递`save`, `insert` 和 `update`参数。

- `data` - 使用`persist`方法传递的其他数据。这个数据可以在订阅者中使用。
- `listeners`: `boolean` - 指示是否为此操作调用监听者和订阅者。默认启用，可以通过在save/remove选项中设置`{listeners：false}`来禁用。
- `transaction`: `boolean` - 默认情况下，启用事务并将持久性操作中的所有查询都包裹在事务中。可以通过在持久性选项中设置`{transaction：false}`来禁用此行为。
- `chunk`: `number` - 中断将执行保存到多个块组中的操作。 例如，如果要保存`100.000`个对象但是在保存它们时遇到问题，可以将它们分成`10`组`10.000`个对象（通过设置`{chunk：10000}`）并分别保存每个组。 当遇到基础驱动程序参数数量限制问题时，需要此选项来执行非常大的插入。
- `reload`: `boolean` - 用于确定是否应在持久性操作期间重新加载正在保留的实体的标志。 它仅适用于不支持`RETURNING/OUTPUT`语句的数据库。 默认情况下启用。

示例:

```typescript
// users包含用user实体数组
userRepository.insert(users, {chunk: users.length / 1000});
```

`RemoveOptions`可以传递`remove`和`delete`参数。

- `data` - 使用`remove`方法传递的其他数据。 这个数据可以在订阅者中使用。
- `listener`: `boolean` - 指示是否为此操作调用监听者和订阅者。默认启用，可以通过在`save/remove`选项中设置`{listeners：false}`来禁用。
- `transaction`: `boolean` - 默认情况下，启用事务并将持久性操作中的所有查询都包裹在事务中。可以通过在持久性选项中设置`{transaction：false}`来禁用此行为。
- `chunk`: `number` - 中断将执行保存到多个块组中的操作。 例如，如果要保存`100.000`个对象但是在保存它们时遇到问题，可以将它们分成`10`组`10.000`个对象（通过设置`{chunk：10000}`）并分别保存每个组。 当遇到基础驱动程序参数数量限制问题时，需要此选项来执行非常大的插入。

示例:

```typescript
// users包含用user实体数组
userRepository.remove(users, {chunk: entities.length / 1000});
```

### 事务

事务是使用`Connection`或`EntityManager`创建的。 例如:

```typescript
import { getConnection } from "typeorm";

await getConnection().transaction(transactionalEntityManager => {});
```

```typescript
import { getManager } from "typeorm";

await getManager().transaction(transactionalEntityManager => {});
```

你想要在事务中运行的所有内容都必须在回调中执行：

```typescript
import { getManager } from "typeorm";

await getManager().transaction(async transactionalEntityManager => {
  await transactionalEntityManager.save(users);
  await transactionalEntityManager.save(photos);
  // ...
});
```

当在事务中工作时需要**总是**使用提供的实体管理器实例 - `transactionalEntityManager`。 如果你使用全局管理器（来自`getManager`或来自连接的 `manager` 可能会遇到一些问题。 你也不能使用使用全局管理器或连接的类来执行查询。**必须**使用提供的事务实体管理器执行所有操作。

#### 指定隔离级别

指定事务的隔离级别可以通过将其作为第一个参数提供来完成：

```typescript
import { getManager } from "typeorm";

await getManager().transaction("SERIALIZABLE", transactionalEntityManager => {});
```

隔离级别实现与所有数据库不相关。

以下数据库驱动程序支持标准隔离级别（`READ UNCOMMITTED`，`READ COMMITTED`，`REPEATABLE READ`，`SERIALIZABLE`）：

- MySQL
- Postgres
- SQL Server

**SQlite**将事务默认为`SERIALIZABLE`，但如果启用了*shared cache mode*，则事务可以使用`READ UNCOMMITTED`隔离级别。

**Oracle**仅支持`READ COMMITTED`和`SERIALIZABLE`隔离级别。

#### 使用 QueryRunner 创建和控制单个数据库连接的状态

`QueryRunner`提供单个数据库连接。 使用查询运行程序组织事务。 单个事务只能在单个查询运行器上建立。 你可以手动创建查询运行程序实例，并使用它来手动控制事务状态。 例如：

```typescript
import { getConnection } from "typeorm";

// 获取连接并创建新的queryRunner
const connection = getConnection();
const queryRunner = connection.createQueryRunner();

// 使用我们的新queryRunner建立真正的数据库连
await queryRunner.connect();

// 现在我们可以在queryRunner上执行任何查询，例如：
await queryRunner.query("SELECT * FROM users");

// 我们还可以访问与queryRunner创建的连接一起使用的实体管理器：
const users = await queryRunner.manager.find(User);

// 开始事务：
await queryRunner.startTransaction();

try {
  // 对此事务执行一些操作：
  await queryRunner.manager.save(user1);
  await queryRunner.manager.save(user2);
  await queryRunner.manager.save(photos);

  // 提交事务：
  await queryRunner.commitTransaction();
} catch (err) {
  // 有错误做出回滚更改
  await queryRunner.rollbackTransaction();
}
```

在`QueryRunner`中有 3 种控制事务的方法：

- `startTransaction` - 启动一个新事务。
- `commitTransaction` - 提交所有更改。
- `rollbackTransaction` - 回滚所有更改。

### Query Builder查询

`QueryBuilder`是 `TypeORM` 最强大的功能之一 ，它允许你使用优雅便捷的语法构建 `SQL` 查询，执行并获得自动转换的实体。

`QueryBuilder`的简单示例:

```typescript
const firstUser = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .where("user.id = :id", { id: 1 })
  .getOne();
```

它将生成以下 `SQL` 查询：

```sql
SELECT
    user.id as userId,
    user.firstName as userFirstName,
    user.lastName as userLastName
FROM users user
WHERE user.id = 1
```

然后返回一个 `User` 实例:

```json
User {
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}
```

#### 创建和使用

有几种方法可以创建`Query Builder`：

- 使用 `connection`:

  ```typescript
  import { getConnection } from "typeorm";
  
  const user = await getConnection()
    .createQueryBuilder()
    .select("user")
    .from(User, "user")
    .where("user.id = :id", { id: 1 })
    .getOne();
  ```

  

- 使用 `entity manager`:

  ```typescript
  import { getManager } from "typeorm";
  
  const user = await getManager()
    .createQueryBuilder(User, "user")
    .where("user.id = :id", { id: 1 })
    .getOne();
  ```

  

- 使用 `repository`:

  ```typescript
  import { getRepository } from "typeorm";
  
  const user = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne();
  ```

  

有 `5` 种不同的`QueryBuilder`类型可用：

- `SelectQueryBuilder` - 用于构建和执行`SELECT`查询。 例如：

  ```typescript
  import { getConnection } from "typeorm";
  
  const user = await getConnection()
    .createQueryBuilder()
    .select("user")
    .from(User, "user")
    .where("user.id = :id", { id: 1 })
    .getOne();
  ```

  

- `InsertQueryBuilder` - 用于构建和执行`INSERT`查询。 例如：

  ```typescript
  import { getConnection } from "typeorm";
  
  await getConnection()
    .createQueryBuilder()
    .insert()
    .into(User)
    .values([{ firstName: "Timber", lastName: "Saw" }, { firstName: "Phantom", lastName: "Lancer" }])
    .execute();
  ```

  

- `UpdateQueryBuilder` - 用于构建和执行`UPDATE`查询。 例如：

  ```typescript
  import { getConnection } from "typeorm";
  
  await getConnection()
    .createQueryBuilder()
    .update(User)
    .set({ firstName: "Timber", lastName: "Saw" })
    .where("id = :id", { id: 1 })
    .execute();
  ```

  

- `DeleteQueryBuilder` - 用于构建和执行`DELETE`查询。 例如：

  ```typescript
  import { getConnection } from "typeorm";
  
  await getConnection()
    .createQueryBuilder()
    .delete()
    .from(User)
    .where("id = :id", { id: 1 })
    .execute();
  ```

  

- `RelationQueryBuilder` - 用于构建和执行特定于关系的操作[`TBD`]。

你可以在其中切换任何不同类型的查询构建器，一旦执行，则将获得一个新的查询构建器实例（与所有其他方法不同）。

#### 获取值

要从数据库中获取单个结果，例如通过 `id` 或 `name` 获取用户，必须使用`getOne`：

```typescript
const timber = await getRepository(User)
  .createQueryBuilder("user")
  .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber" })
  .getOne();
```

要从数据库中获取多个结果，例如，要从数据库中获取所有用户，请使用`getMany`：

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .getMany();
```

使用查询构建器查询可以获得两种类型的结果：`entities` 或 `raw results`。 大多数情况下，你只需要从数据库中选择真实实体，例如 `users`。 为此，你可以使用`getOne`和`getMany`。 但有时你需要选择一些特定的数据，比方说所有`sum of all user photos`。 此数据不是实体，它称为原始数据。 要获取原始数据，请使用`getRawOne`和`getRawMany`。 例如：

```typescript
const { sum } = await getRepository(User)
  .createQueryBuilder("user")
  .select("SUM(user.photosCount)", "sum")
  .where("user.id = :id", { id: 1 })
  .getRawOne();
```

```typescript
const photosSums = await getRepository(User)
  .createQueryBuilder("user")
  .select("user.id")
  .addSelect("SUM(user.photosCount)", "sum")
  .where("user.id = :id", { id: 1 })
  .getRawMany();

// 结果会像这样: [{ id: 1, sum: 25 }, { id: 2, sum: 13 }, ...]
```

#### 别名

我们使用`createQueryBuilder（"user"）`。 但什么是`user`？ 它只是一个常规的 `SQL` 别名。 我们在任何地方都使用别名，除非我们处理选定的数据。

`createQueryBuilder("user")` 相当于：

```typescript
createQueryBuilder()
  .select("user")
  .from(User, "user");
```

这会生成以下 `sql` 查询：

```sql
SELECT ... FROM users user
```

在这个 `SQL` 查询中，`users`是表名，`user`是我们分配给该表的别名。

稍后我们使用此别名来访问表：

```typescript
createQueryBuilder()
  .select("user")
  .from(User, "user")
  .where("user.name = :name", { name: "Timber" });
```

以上代码会生成如下 `SQL` 语句：

```sql
SELECT ... FROM users user WHERE user.name = 'Timber'
```

看到了吧，我们使用了在创建查询构建器时分配的`user`别名来使用 `users` 表。

一个查询构建器不限于一个别名，它们可以有多个别名。 每个选择都可以有自己的别名，你可以选择多个有自己别名的表，你可以使用自己的别名连接多个表。 你也可以使用这些别名来访问选择的表（或正在选择的数据）。

#### 使用参数来转义数据

我们使用了`where("user.name = :name", { name: "Timber" })`. `{name：“Timber”}`代表什么？ 这是我们用来阻止 `SQL注入`的参数。 我们可以写：`where（"user.name ='"+ name +"'）`，但是这不安全，因为有可能被 `SQL注入`。 安全的方法是使用这种特殊的语法：`where（"user.name = :name"，{name:"Timber"}）`，其中`name`是参数名，值在对象中指定： `{name:"Timber"}`。

```typescript
.where("user.name = :name", { name: "Timber" })
```

是下面的简写：

```typescript
.where("user.name = :name")
.setParameter("name", "Timber")
```

注意：不要在查询构建器中为不同的值使用相同的参数名称。如果多次设置则后值将会把前面的覆盖。

还可以提供一组值，并使用特殊的扩展语法将它们转换为`SQL`语句中的值列表：

```typescript
.where("user.name IN (:...names)", { names: [ "Timber", "Cristal", "Lina" ] })
```

该语句将生成：

```sql
WHERE user.name IN ('Timber', 'Cristal', 'Lina')
```

#### 添加WHERE表达式

添加 `WHERE` 表达式就像：

```typescript
createQueryBuilder("user").where("user.name = :name", { name: "Timber" });
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user WHERE user.name = 'Timber'
```

你可以将 `AND` 添加到现有的 `WHERE` 表达式中：

```typescript
createQueryBuilder("user")
  .where("user.firstName = :firstName", { firstName: "Timber" })
  .andWhere("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user WHERE user.firstName = 'Timber' AND user.lastName = 'Saw'
```

你也可以添加 `OR` 添加到现有的 `WHERE` 表达式中：

```typescript
createQueryBuilder("user")
  .where("user.firstName = :firstName", { firstName: "Timber" })
  .orWhere("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user WHERE user.firstName = 'Timber' OR user.lastName = 'Saw'
```

你可以使用`Brackets`将复杂的`WHERE`表达式添加到现有的`WHERE`中：

```typescript
createQueryBuilder("user")
    .where("user.registered = :registered", { registered: true })
    .andWhere(new Brackets(qb => {
        qb.where("user.firstName = :firstName", { firstName: "Timber" })
          .orWhere("user.lastName = :lastName", { lastName: "Saw" })
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user WHERE user.registered = true AND (user.firstName = 'Timber' OR user.lastName = 'Saw')
```

你可以根据需要组合尽可能多的`AND`和`OR`表达式。 如果你多次使用`.where`，你将覆盖所有以前的`WHERE`表达式。 注意：小心`orWhere` - 如果你使用带有`AND`和`OR`表达式的复杂表达式，请记住他们将无限制的叠加。 有时你只需要创建一个 `where` 字符串，避免使用`orWhere`。

#### 添加HAVING表达式

添加`HAVING`表达式很简单：

```typescript
createQueryBuilder("user").having("user.name = :name", { name: "Timber" });
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user HAVING user.name = 'Timber'
```

你可以添加 `AND` 到已经存在的 `HAVING` 表达式中：

```typescript
createQueryBuilder("user")
  .having("user.firstName = :firstName", { firstName: "Timber" })
  .andHaving("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user HAVING user.firstName = 'Timber' AND user.lastName = 'Saw'
```

你可以添加 `OR` 到已经存在的 `HAVING` 表达式中：

```typescript
createQueryBuilder("user")
  .having("user.firstName = :firstName", { firstName: "Timber" })
  .orHaving("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user HAVING user.firstName = 'Timber' OR user.lastName = 'Saw'
```

你可以根据需要组合尽可能多的`AND`和`OR`表达式。 如果使用多个`.having`，后面的将覆盖所有之前的`HAVING`表达式。

#### 添加ORDER BY表达式

添加 `ORDER BY` 很简单：

```typescript
createQueryBuilder("user").orderBy("user.id");
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user ORDER BY user.id
```

你可以将排序方向从升序更改为降序（或反之亦然）：

```typescript
createQueryBuilder("user").orderBy("user.id", "DESC");

createQueryBuilder("user").orderBy("user.id", "ASC");
```

也可以添加多个排序条件：

```typescript
createQueryBuilder("user")
  .orderBy("user.name")
  .addOrderBy("user.id");
```

还可以使用排序字段作为一个 `map`：

```typescript
createQueryBuilder("user").orderBy({
  "user.name": "ASC",
  "user.id": "DESC"
});
```

如果你使用了多个`.orderBy`，后面的将覆盖所有之前的`ORDER BY`表达式。

#### 添加GROUP BY表达式

添加 `GROUP BY` 表达式很简单：

```typescript
createQueryBuilder("user").groupBy("user.id");
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user GROUP BY user.id
```

如果要使用更多 `group-by`, 则可以使用 `addGroupBy`:

```typescript
createQueryBuilder("user")
  .groupBy("user.name")
  .addGroupBy("user.id");
```

如果使用了多个`.groupBy` ，则后面的将会覆盖之前所有的 `ORDER BY` 表达式。

#### 添加LIMIT表达式

添加 `LIMIT` 表达式很简单：

```typescript
createQueryBuilder("user").limit(10);
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user LIMIT 10
```

生成的 `SQL` 查询取决于数据库的类型（`SQL`，`mySQL`，`Postgres` 等）。 注意：如果你使用带有连接或子查询的复杂查询，`LIMIT` 可能无法正常工作。 如果使用分页，建议使用`take`代替。

#### 添加OFFSET表达式

添加`OFFSET`表达式很简单：

```typescript
createQueryBuilder("user").offset(10);
```

将会生成以下 `SQL` 语句：

```sql
SELECT ... FROM users user OFFSET 10
```

生成的 `SQL` 查询取决于数据库的类型（`SQL`，`mySQL`，`Postgres` 等）。 注意：如果你使用带有连接或子查询的复杂查询，`OFFSET` 可能无法正常工作。 如果使用分页，建议使用`skip`代替。

#### 联查

假设有以下实体：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(type => Photo, photo => photo.user)
  photos: Photo[];
}
```

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @ManyToOne(type => User, user => user.photos)
  user: User;
}
```

现在让我们假设你要用用户`"Timber"`加载他所有的` photos`：

```typescript
const user = await createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .where("user.name = :name", { name: "Timber" })
  .getOne();
```

你将会得到以下结果：

```typescript
{
    id: 1,
    name: "Timber",
    photos: [{
        id: 1,
        url: "me-with-chakram.jpg"
    }, {
        id: 2,
        url: "me-with-trees.jpg"
    }]
}
```

你可以看到`leftJoinAndSelect`自动加载了所有 `Timber` 的 `photos`。 第一个参数是你要加载的关系，第二个参数是你为此关系的表分配的别名。 你可以在查询构建器中的任何位置使用此别名。 例如，让我们获得所有未删除的 `Timber` 的 `photos`。

```typescript
const user = await createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .where("user.name = :name", { name: "Timber" })
  .andWhere("photo.isRemoved = :isRemoved", { isRemoved: false })
  .getOne();
```

将会生成以下 `SQL` 查询：

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id
    WHERE user.name = 'Timber' AND photo.isRemoved = FALSE
```

你还可以向连接表达式添加条件，而不是使用`"where"`：

```typescript
const user = await createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", { isRemoved: false })
  .where("user.name = :name", { name: "Timber" })
  .getOne();
```

这将生成以下 `sql` 查询：

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'
```

#### 内联和左联

如果你想使用`INNER JOIN`而不是`LEFT JOIN`，只需使用`innerJoinAndSelect`：

```typescript
const user = await createQueryBuilder("user")
  .innerJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", { isRemoved: false })
  .where("user.name = :name", { name: "Timber" })
  .getOne();
```

将会生成:

```sql
SELECT user.*, photo.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'
```

`LEFT JOIN`和`INNER JOIN`之间的区别在于，如果没有任何 `photos`，`INNER JOIN`将不会返回 `user`。 即使没有 `photos`，`LEFT JOIN`也会返回 `user`。 

#### 不使用条件的联查

你可以在不使用条件的情况下联查数据。 要做到这一点，使用`leftJoin`或`innerJoin`：

```typescript
const user = await createQueryBuilder("user")
  .innerJoin("user.photos", "photo")
  .where("user.name = :name", { name: "Timber" })
  .getOne();
```

将会生成如下 `SQL` 语句：

```sql
SELECT user.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id
    WHERE user.name = 'Timber'
```

这将会返回 `Timber` ，如果他有 `photos`，但是并不会返回他的 `photos`。

#### 联查任何实体或表

你不仅能联查关系，还能联查任何其他实体或表。

例如：

```typescript
const user = await createQueryBuilder("user")
  .leftJoinAndSelect(Photo, "photo", "photo.userId = user.id")
  .getMany();
```

```typescript
const user = await createQueryBuilder("user")
  .leftJoinAndSelect("photos", "photo", "photo.userId = user.id")
  .getMany();
```

#### 联查和映射功能

将`profilePhoto`添加到`User`实体，你可以使用`QueryBuilder`将任何数据映射到该属性：

```typescript
export class User {
  /// ...
  profilePhoto: Photo;
}
```

```typescript
const user = await createQueryBuilder("user")
  .leftJoinAndMapOne("user.profilePhoto", "user.photos", "photo", "photo.isForProfile = TRUE")
  .where("user.name = :name", { name: "Timber" })
  .getOne();
```

这将加载 `Timber` 的个人资料照片并将其设置为`user.profilePhoto`。 如果要加载并映射单个实体，请使用`leftJoinAndMapOne`。 如果要加载和映射多个实体，请使用`leftJoinAndMapMany`。

#### 获取生成的sql查询语句

有时你可能想要获取`QueryBuilder`生成的 `SQL` 查询。 为此，请使用`getSql`：

```typescript
const sql = createQueryBuilder("user")
  .where("user.firstName = :firstName", { firstName: "Timber" })
  .orWhere("user.lastName = :lastName", { lastName: "Saw" })
  .getSql();
```

出于调试目的，你也可以使用`printSql`：

```typescript
const users = await createQueryBuilder("user")
  .where("user.firstName = :firstName", { firstName: "Timber" })
  .orWhere("user.lastName = :lastName", { lastName: "Saw" })
  .printSql()
  .getMany();
```

此查询将返回 `users` 并将使用的 `sql` 语句打印到控制台。

#### 获得原始结果

使用选择查询构建器可以获得两种类型的结果：`entities` 和 `raw results`。 大多数情况下，你只需要从数据库中选择真实实体，例如 `users`。 为此，你可以使用`getOne`和`getMany`。 但是，有时需要选择特定数据，例如 `sum of all user photos`。 这些数据不是实体，它被称为原始数据。 要获取原始数据，请使用`getRawOne`和`getRawMany`。 例如：

```typescript
const { sum } = await getRepository(User)
  .createQueryBuilder("user")
  .select("SUM(user.photosCount)", "sum")
  .where("user.id = :id", { id: 1 })
  .getRawOne();
```

```typescript
const photosSums = await getRepository(User)
  .createQueryBuilder("user")
  .select("user.id")
  .addSelect("SUM(user.photosCount)", "sum")
  .where("user.id = :id", { id: 1 })
  .getRawMany();

// 结果将会像这样: [{ id: 1, sum: 25 }, { id: 2, sum: 13 }, ...]
```

#### 流数据

你可以使用`stream`来返回流。 `Streaming` 返回原始数据，必须手动处理实体转换：

```typescript
const stream = await getRepository(User)
  .createQueryBuilder("user")
  .where("user.id = :id", { id: 1 })
  .stream();
```

#### 使用分页

大多数情况下，在开发应用程序时，你可能需要分页功能。 如果你的应用程序中有分页，`page slider` 或无限滚动组件，则使用此选项。

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .take(10)
  .getMany();
```

将会返回前 `10` 个 `user` 的 `photos`。

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .skip(10)
  .getMany();
```

将返回除了前 `10` 个 `user` 以外的所有 `user` 的 `photos`。

你可以组合这些方法：

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .skip(5)
  .take(10)
  .getMany();
```

这将跳过前 `5` 个 `users`，并获取他们之后的 `10` 个 `user`。

`take`和`skip`可能看起来像我们正在使用`limit`和`offset`，但它们不是。 一旦你有更复杂的连接或子查询查询，`limit`和`offset`可能无法正常工作。 使用`take`和`skip`可以防止这些问题。

#### 加锁

`QueryBuilder` 支持 `optimistic` 和 `pessimistic` 锁定。 要使用 `pessimistic 读锁定`，请使用以下方式：

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .setLock("pessimistic_read")
  .getMany();
```

要使用 `pessimistic 写锁定`，请使用以下方式：

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .setLock("pessimistic_write")
  .getMany();
```

要使用 `optimistic 读锁定`，请使用以下方式：

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .setLock("optimistic", existUser.version)
  .getMany();
```

要使用 `dirty 读锁定`，请使用以下方式：

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("dirty_read")
    .getMany();
```

`Optimistic` 锁定与`@Version`和`@UpdatedDate`装饰器一起使用。

#### 查询部分字段

如果只想选择实体的某些属性，可以使用以下语法：

```typescript
const users = await getRepository(User)
  .createQueryBuilder("user")
  .select(["user.id", "user.name"])
  .getMany();
```

这只会选择`User`的`id`和`name`。

#### 使用子查询

你可以轻松创建子查询。 `FROM`，`WHERE`和`JOIN`表达式都支持子查询。 例如：

```typescript
const qb = await getRepository(Post).createQueryBuilder("post");
const posts = qb
  .where(
    "post.title IN " +
      qb
        .subQuery()
        .select("user.name")
        .from(User, "user")
        .where("user.registered = :registered")
        .getQuery()
  )
  .setParameter("registered", true)
  .getMany();
```

使用更优雅的方式来做同样的事情：

```typescript
const posts = await connection
  .getRepository(Post)
  .createQueryBuilder("post")
  .where(qb => {
    const subQuery = qb
      .subQuery()
      .select("user.name")
      .from(User, "user")
      .where("user.registered = :registered")
      .getQuery();
    return "post.title IN " + subQuery;
  })
  .setParameter("registered", true)
  .getMany();
```

或者，你可以创建单独的查询构建器并使用其生成的 `SQL`：

```typescript
const userQb = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .select("user.name")
  .where("user.registered = :registered", { registered: true });

const posts = await connection
  .getRepository(Post)
  .createQueryBuilder("post")
  .where("post.title IN (" + userQb.getQuery() + ")")
  .setParameters(userQb.getParameters())
  .getMany();
```

你可以在`FROM`中创建子查询，如下所示：

```typescript
const userQb = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .select("user.name", "name")
  .where("user.registered = :registered", { registered: true });

const posts = await connection
  .createQueryBuilder()
  .select("user.name", "name")
  .from("(" + userQb.getQuery() + ")", "user")
  .setParameters(userQb.getParameters())
  .getRawMany();
```

或使用更优雅的语法：

```typescript
const posts = await connection
  .createQueryBuilder()
  .select("user.name", "name")
  .from(subQuery => {
    return subQuery
      .select("user.name", "name")
      .from(User, "user")
      .where("user.registered = :registered", { registered: true });
  }, "user")
  .getRawMany();
```

如果想添加一个子查询做为`"second from"`，请使用`addFrom`。

你也可以在`SELECT`语句中使用子查询：

```typescript
const posts = await connection
  .createQueryBuilder()
  .select("post.id", "id")
  .addSelect(subQuery => {
    return subQuery
      .select("user.name", "name")
      .from(User, "user")
      .limit(1);
  }, "name")
  .from(Post, "post")
  .getRawMany();
```

#### 隐藏列

如果要查询的模型具有`"select：false"`的列，则必须使用`addSelect`函数来从列中检索信息。

假设你有以下实体：

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ select: false })
  password: string;
}
```

使用标准的`find`或查询，你将不会接收到模型的`password`属性。 但是，如果执行以下操作：

```typescript
const users = await connection
  .getRepository(User)
  .createQueryBuilder()
  .select("user.id", "id")
  .addSelect("user.password")
  .getMany();
```

你将在查询中获得属性`password`。

### Query builder插入

你可以使用`QueryBuilder`创建`INSERT`查询。 例如：

```typescript
import { getConnection } from "typeorm";

await getConnection()
  .createQueryBuilder()
  .insert()
  .into(User)
  .values([{ firstName: "Timber", lastName: "Saw" }, { firstName: "Phantom", lastName: "Lancer" }])
  .execute();
```

就性能而言，这是向数据库中插入实体的最有效方法。 你也可以通过这种方式执行批量插入。

#### 原始SQL支持

在某些情况下需要执行函数`SQL`查询时：

```typescript
import {getConnection} from "typeorm";

await getConnection()
    .createQueryBuilder()
    .insert()
    .into(User)
    .values({ 
        firstName: "Timber", 
        lastName: () => "CONCAT('S', 'A', 'W')"
    })
    .execute();
```

此语法不会对值进行转义，你需要自己处理转义。

### Query Builder更新

你可以使用`QueryBuilder`创建`UPDATE`查询。 例如：

```typescript
import { getConnection } from "typeorm";

await getConnection()
  .createQueryBuilder()
  .update(User)
  .set({ firstName: "Timber", lastName: "Saw" })
  .where("id = :id", { id: 1 })
  .execute();
```

就性能而言，这是更新数据库中的实体的最有效方法。

#### 原始SQL支持

在某些情况下需要执行函数`SQL`查询时：

```typescript
import {getConnection} from "typeorm";
await getConnection()
   .createQueryBuilder()
   .update(User)
   .set({ 
       firstName: "Timber", 
       lastName: "Saw",
       age: () => "'age' + 1"
   })
   .where("id = :id", { id: 1 })
   .execute();
```

此语法不会对值进行转义，你需要自己处理转义。

### Query Builder 删除

你可以使用`QueryBuilder`创建`DELETE`查询。 例如：

```typescript
import { getConnection } from "typeorm";

await getConnection()
  .createQueryBuilder()
  .delete()
  .from(User)
  .where("id = :id", { id: 1 })
  .execute();
```

就性能而言，这是删除数据库中的实体的最有效方法。

### 与Relations结合

`RelationQueryBuilder`是`QueryBuilder`的一种允许你使用关系来查询的特殊类型。 通过使用你可以在数据库中将实体彼此绑定，而无需加载任何实体，或者可以轻松地加载相关实体。 例如：

例如，我们有一个`Post`实体，它与`Category`有一个多对多的关系，叫做`categories`。 让我们为这种多对多关系添加一个新 category：

```typescript
import { getConnection } from "typeorm";

await getConnection()
  .createQueryBuilder()
  .relation(Post, "categories")
  .of(post)
  .add(category);
```

这段代码相当于：

```typescript
import { getManager } from "typeorm";

const postRepository = getRepository(Post);
const post = await postRepository.findOne(1, { relations: ["categories"] });
post.categories.push(category);
await postRepository.save(post);
```

但是这样使用第一种方式效率更高，因为它执行的操作数量最少，并且绑定数据库中的实体，这比每次都调用`save`这种笨重的方法简化了很多。

此外，这种方法的另一个好处是不需要在 `pushing` 之前加载每个相关实体。 例如，如果你在一个 `post` 中有一万个 `categories`，那么在此列表中添加新 `posts` 可能会产生问题，因为执行此操作的标准方法是加载包含所有一万个 `categories` 的 `post`，`push` 一个新 `category` 然后保存。 这将会导致非常高的性能成本，而且基本上不适用于生产环境。 但是，使用`RelationQueryBuilder`则解决了这个问题。

此外，当进行绑定时，可以不需要使用实体，只需要使用实体 `ID` 即可。 例如，让我们在 `id` 为 `1` 的 `post` 中添加 `id = 3` 的 `category`：

```typescript
import { getConnection } from "typeorm";

await getConnection()
  .createQueryBuilder()
  .relation(Post, "categories")
  .of(1)
  .add(3);
```

如果你使用了复合主键，则必须将它们作为 `id` 映射传递，例如：

```typescript
import { getConnection } from "typeorm";

await getConnection()
  .createQueryBuilder()
  .relation(Post, "categories")
  .of({ firstPostId: 1, secondPostId: 3 })
  .add({ firstCategoryId: 2, secondCategoryId: 4 });
```

以按照添加实体的方式删除实体：

```typescript
import { getConnection } from "typeorm";

// 此代码从给定的post中删除一个category
await getConnection()
  .createQueryBuilder()
  .relation(Post, "categories")
  .of(post) // 也可以使用post id
  .remove(category); // 也可以只使用category ID
```

添加和删除相关实体针对`多对多`和`一对多`关系。 对于`一对一`和`多对一`关系，请使用`set`代替：

```typescript
import { getConnection } from "typeorm";

// 此代码set给定post的category
await getConnection()
  .createQueryBuilder()
  .relation(Post, "categories")
  .of(post) // 也可以使用post id
  .set(category); // 也可以只使用category ID
```

如果要取消设置关系（将其设置为 `null`），只需将`null`传递给`set`方法：

```typescript
import { getConnection } from "typeorm";

// 此代码取消设置给定post的category
await getConnection()
  .createQueryBuilder()
  .relation(Post, "categories")
  .of(post) // 也可以使用post id
  .set(null);
```

除了更新关系外，关系查询构建器还允许你加载关系实体。 例如，假设在`Post`实体内部，我们有多对多的`categories`关系和多对一的`user`关系，为加载这些关系，你可以使用以下代码：

```typescript
import { getConnection } from "typeorm";

const post = await getConnection().manager.findOne(Post, 1);

post.categories = await getConnection()
  .createQueryBuilder()
  .relation(Post, "categories")
  .of(post) // 也可以使用post id
  .loadMany();

post.author = await getConnection()
  .createQueryBuilder()
  .relation(User, "user")
  .of(post) // 也可以使用post id
  .loadOne();
```
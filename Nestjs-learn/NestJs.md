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


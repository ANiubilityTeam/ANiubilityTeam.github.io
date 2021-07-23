---
title: Koa+MySQL+TypeORM项目搭建
author: piers
tags:
  - Koa
  - TypeORM
  - MySQL
categories:
  - 前端工程化
description: Koa作为Express原班人马打造的新生代 Node.js Web 框架，自从发布以来就备受瞩目。凭借精巧的 “洋葱模型” 和对Promise以及async/await异步编程的完全支持，Koa框架自从诞生以来就吸引了无数Node爱好者。然而Koa本身只是一个简单的中间件框架，要想实现一个足够复杂的 Web 应用还需要很多周边生态支持...
date: 2021-06-24
---

## 简介

`Koa` 作为 `Express` 原班人马打造的新生代 Node.js Web 框架，自从发布以来就备受瞩目。凭借精巧的 “洋葱模型” 和对 `Promise` 以及 `async/await` 异步编程的完全支持，`Koa` 框架自从诞生以来就吸引了无数 Node 爱好者。然而 `Koa` 本身只是一个简单的中间件框架，要想实现一个足够复杂的 Web 应用还需要很多周边生态支持。

## 准备

### 环境准备

- Node.js：10.x 及以上
- npm：6.x 及以上
- Koa：2.x
- MySQL：推荐稳定的 5.7 版本及以上
- TypeORM：0.2.x

### 学习目标

- 如何编写 `Koa` 中间件
- 通过 `@koa/router` 实现路由配置
- 通过 `TypeORM` 连接和读写 `MySQL` 数据库（其他数据库都类似）
- 了解 `JWT` 鉴权的原理，并动手实现
- 掌握 `Koa` 的错误处理机制

### 初始代码

初始化代码的仓库和分支

`git clone -b init-porject https://github.com/ANiubilityTeam/koa.git`

安装依赖 启动项目

`npm install` `npm start`

包含了一个最简单的服务

```javascript
// src/server.ts

import Koa from "koa";
import cors from "@koa/cors";
import bodyParser from "koa-bodyparser";
// 初始化 Koa 应用实例
const app = new Koa();
// 注册中间件
app.use(cors());
app.use(bodyParser());
// 响应用户请求
app.use((ctx) => {
  ctx.body = "Hello Koa";
});
// 运行服务器
app.listen(3000);
```

启动后浏览器访问 localhost:3000,效果应该是这样的
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21f791a609664f79a7d8c8a250296aa4~tplv-k3u1fbpfcp-watermark.image)

或者 `$ curl localhost:3000`

得到 `Hello Koat`

## Koa 中间件

严格意义上来说，Koa 只是一个中间件框架，正如它的介绍所说：

> Expressive middleware for node.js using ES2017 async functions.（通过 ES2017 async 函数编写富有表达力的 Node.js 中间件）

| Feature       | Koa | Express | connect |
| ------------- | --- | ------- | ------- |
| middleware    | Y   | Y       | Y       |
| route         |     | Y       |         |
| template      |     | Y       |         |
| sending files |     | Y       |         |
| jsonp         |     | Y       |         |

可以看出来 Koa 对标的是[connect](https://github.com/senchalabs/connect)（Express 底层的中间件层），而不包含 Express 所拥有的其他功能，例如路由、模板引擎、发送文件等。

### Express 使用的中间件模型

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18d5ac5f44484859abf3a4c51a7d2721~tplv-k3u1fbpfcp-watermark.image)
在 Express 中，请求（Request）直接依次贯穿各个中间件，最后通过请求处理函数返回响应（Response），非常简单。

所以 Express 中请求处理函数是这样的, 两个参数分别对应请求对象（Request）和响应对象（Response)

```javascript
function handler(req, res) {
  res.send("Hello Express");
}
```

### Koa 使用的中间件模型 - 洋葱模型

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/892c05eecb054fdcb47d8f771619ff3c~tplv-k3u1fbpfcp-watermark.image)
在 Koa 中，中间件不像 Express 中间件那样在请求通过了之后就完成了自己的使命；相反，中间件的执行清晰地分为两个阶段。

就像是一个洋葱，一层包裹一层
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84ffc59d814549b48fc8f717a2036d4a~tplv-k3u1fbpfcp-watermark.image)

所以 Koa 中，请求处理函数是这样的，只有一个参数 ctx （Context，上下文），然后只需向上下文对象写入相关的属性即可（例如这里就是写入到返回数据 body 中）

```js
function handler(ctx) {
  ctx.body = "Hello Koa";
}
```

### Koa 中间件的定义

中间件其实也是一个函数

```js
async function middleware(ctx, next) {
  // 第一阶段
  await next();
  // 第二阶段
}
```

第一个参数就是 Koa Context，也就是上图中贯穿所有中间件和请求处理函数的绿色箭头所传递的内容，里面封装了请求体和响应体（实际上还有其他属性，但这里暂时不讲），分别可以通过 ctx.request 和 ctx.response 来获取，以下是一些常用的属性：

```js
ctx.url; // 相当于 ctx.request.url
ctx.body; // 相当于 ctx.response.body
ctx.status; // 相当于 ctx.response.status
```

> 关于所有请求和响应上面的属性及其别称，请参考 [Context API 文档](https://github.com/koajs/koa/blob/master/docs/api/context.md)

中间件的第二个参数便是 next 函数，用来把控制权转交给下一个中间件。但是它跟 Express 的 next 函数本质的区别在于，Koa 的`next`函数返回的是一个 Promise，在这个 Promise 进入完成状态（Fulfilled）后，就会去执行中间件中第二阶段的代码。

### 实现一个日志中间件

```js
// src/logger.ts

import { Context } from "koa";

export function logger() {
  return async (ctx: Context, next: () => Promise<void>) => {
    const start = Date.now();
    await next();
    const ms = Date.now() - start;
    console.log(`${ctx.method} ${ctx.url} ${ctx.status} - ${ms}ms`);
  };
}
```

在这个中间件的第一阶段，我们通过 Date.now() 先获取请求进入的时间，然后通过 await next() 让出执行权，等待下游中间件运行结束后，再在第二阶段通过计算 Date.now() 的差值来得出处理请求所用的时间。

> 这里通过两个 Date.now() 之间的差值来计算运行时间其实是不精确的，为了获取更准确的时间，建议使用 process.hrtime() 。

然后我们在 src/server.ts 中把刚才的 logger 中间件通过 app.use 注册进去，代码如下：

```js
// ...
import { logger } from "./logger";

// 初始化 Koa 应用实例
const app = new Koa();

// 注册中间件
app.use(logger());
app.use(cors());
app.use(bodyParser());

// ...
```

然后通过 crul 或者浏览器访问 localhost:3000

终端会打印

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbdba0dff89445f2a45ac1c44c217de9~tplv-k3u1fbpfcp-watermark.image)

## Koa 路由配置

由于 Koa 只是一个中间件框架，所以路由的实现需要独立的 npm 包。首先安装 @koa/router 及其 TypeScript 类型定义

`$ npm install @koa/router`

`$ npm install @types/koa__router -D`

> 有些教程使用 koa-router ，但由于 koa-router 目前处于几乎无人维护的状态，所以我们这里使用维护更积极的 Fork 版本 @koa/router。

### 路由规划

- GET /users ：查询所有的用户
- GET /users/:id ：查询单个用户
- PUT /users/:id ：更新单个用户
- DELETE /users/:id ：删除单个用户
- POST /users/login ：登录（获取 JWT Token）
- POST /users/register ：注册用户

### 创建控制器

在 src 中创建 controllers 目录，用于存放控制器有关的代码。首先是 AuthController

创建 src/controllers/auth.ts ，代码如下：

```js

import { Context } from 'koa';

export default class AuthController {
  public static async login(ctx: Context) {
    ctx.body = 'Login controller';
  }

  public static async register(ctx: Context) {
    ctx.body = 'Register controller';
  }
}
```

然后创建 src/controllers/user.ts，代码如下：

```js

import { Context } from 'koa';

export default class UserController {
  public static async listUsers(ctx: Context) {
    ctx.body = 'ListUsers controller';
  }

  public static async showUserDetail(ctx: Context) {
    ctx.body = `ShowUserDetail controller with ID = ${ctx.params.id}`;
  }

  public static async updateUser(ctx: Context) {
    ctx.body = `UpdateUser controller with ID = ${ctx.params.id}`;
  }

  public static async deleteUser(ctx: Context) {
    ctx.body = `DeleteUser controller with ID = ${ctx.params.id}`;
  }
}
```

### 创建路由

然后我们创建 src/routes.ts，用于把控制器挂载到对应的路由上面

```js
import Router from "@koa/router";

import AuthController from "./controllers/auth";
import UserController from "./controllers/user";

const router = new Router();

// auth 相关的路由
router.post("/auth/login", AuthController.login);
router.post("/auth/register", AuthController.register);

// users 相关的路由
router.get("/users", UserController.listUsers);
router.get("/users/:id", UserController.showUserDetail);
router.put("/users/:id", UserController.updateUser);
router.delete("/users/:id", UserController.deleteUser);

export default router;
```

### 注册路由

将 router 注册为中间件。打开 src/server.ts，修改代码如下

```js
// ...

import router from "./routes";
import { logger } from "./logger";

// 初始化 Koa 应用实例
const app = new Koa();

// 注册中间件
app.use(logger());
app.use(cors());
app.use(bodyParser());

// 响应用户请求
app.use(router.routes()).use(router.allowedMethods());

// 运行服务器
app.listen(3000);
```

调用 router 对象的 routes 方法获取到对应的 Koa 中间件，还调用了 allowedMethods 方法注册了 HTTP 方法检测的中间件，这样当用户通过不正确的 HTTP 方法访问 API 时，就会自动返回 405 Method Not Allowed 状态码。

我们通过 Curl 来测试路由

```BASH
$ curl localhost:3000/hello
Not Found
$ curl localhost:3000/auth/register
Method Not Allowed
$ curl -X POST localhost:3000/auth/register
Register controller
$ curl -X POST localhost:3000/auth/login
Login controller
$ curl localhost:3000/users
ListUsers controller
$ curl localhost:3000/users/123
ShowUserDetail controller with ID = 123
$ curl -X PUT localhost:3000/users/123
UpdateUser controller with ID = 123
$ curl -X DELETE localhost:3000/users/123
DeleteUser controller with ID = 123
```

## 接入 MySQL

Koa 本身是一个中间件框架，理论上可以接入任何类型的数据库，这里我们选择流行的关系型数据库 MySQL。并且，由于我们使用了 TypeScript 开发，因此这里使用为 TS 量身打造的 [ORM](http://www.ruanyifeng.com/blog/2019/02/orm-tutorial.html) 库 TypeORM。

### 准备数据库

- 官网下载安装包，这里是[下载地址](https://dev.mysql.com/downloads/mysql/)
- 安装好后，启动数据库，macOS 是在 系统偏好设置 - MySQL - Start MySQL Server
- 连接数据库`mysql -u root -p`
- 输入安装数据库时设置的密码
- 创建数据库`CREATE DATABASE koa;`
- 创建用户并授予权限`CREATE USER 'user'@'localhost' IDENTIFIED BY 'pass';` `GRANT ALL PRIVILEGES ON koa.* TO 'user'@'localhost';`
- 处理 MySQL 8.0 版本的认证协议问题`ALTER USER 'user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'pass';` `flush privileges;`

### 配置和连接 TypeORM

安装相关的 npm 包，分别是 MySQL 驱动、TypeORM 及 reflect-metadata（反射 API 库，用于 TypeORM 推断模型的元数据）

`$ npm install mysql typeorm reflect-metadata`

然后在项目根目录创建 ormconfig.json ，TypeORM 会读取这个数据库配置进行连接，代码如下

```JS
{
  "type": "mysql",
  "host": "localhost",
  "port": 3306,
  "username": "user",
  "password": "pass",
  "database": "koa",
  "synchronize": true,
  "entities": ["src/entity/*.ts"],
  "cli": {
    "entitiesDir": "src/entity"
  }
}
```

- database 就是我们刚刚创建的 koa 数据库
- synchronize 设为 true 能够让我们每次修改模型定义后都能自动同步到数据库（如果你接触过其他的 ORM 库，其实就是自动数据迁移）
- entities 字段定义了模型文件的路径，我们马上就来创建

接着修改 src/server.ts，在其中连接数据库，代码如下

```js
import Koa from "koa";
import cors from "@koa/cors";
import bodyParser from "koa-bodyparser";
import { createConnection } from "typeorm";
import "reflect-metadata";

import router from "./routes";
import { logger } from "./logger";

// 初始化 Koa 应用实例
createConnection()
  .then(() => {
    const app = new Koa();
    // 注册中间件
    app.use(logger());
    app.use(cors());
    app.use(bodyParser());
    // 响应用户请求
    app.use(router.routes()).use(router.allowedMethods());
    // 运行服务器
    app.listen(3000);
  })
  .catch((err: string) => console.log("TypeORM connection error:", err));
```

### 创建数据模型定义

创建 src/entity/user.ts , 用于存放数据模型定义文件,代表用户模型

```js
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ select: false })
  password: string;

  @Column()
  email: string;
}
```

TypeORM 通过[装饰器](https://www.tslang.cn/docs/handbook/decorators.html)这种优雅的方式来将我们的 User 类映射到数据库中的表。这里我们使用了三个装饰器：

- Entity 用于装饰整个类，使其变成一个数据库模型
- Column 用于装饰类的某个属性，使其对应于数据库表中的一列，可提供一系列选项参数，例如我们给 password 设置了 select: false ，使得这个字段在查询时默认不被选中
- PrimaryGeneratedColumn 则是装饰主列，它的值将自动生成
  > 关于 TypeORM 所有的装饰器定义及其详细使用，请参考其[装饰器文档](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/decorator-reference.md)。

### 控制器中操作数据库

然后就可以在 Controller 中进行数据的增删改查操作了。首先我们打开 src/controllers/user.ts ，实现所有 Controller 的逻辑

```js
import { Context } from 'koa';
import { getManager } from 'typeorm';
import { User } from '../entity/user';

export default class UserController {
  public static async listUsers(ctx: Context) {
    // ctx.body = 'ListUsers controller';
    const userRepository = getManager().getRepository(User);
    const users = await userRepository.find();

    ctx.status = 200;
    ctx.body = users;
  }

  public static async showUserDetail(ctx: Context) {
    // ctx.body = `ShowUserDetail controller with ID = ${ctx.params.id}`;
    const userRepository = getManager().getRepository(User);
    const user = await userRepository.findOne(+ctx.params.id);

    if (user) {
      ctx.status = 200;
      ctx.body = user;
    } else {
      ctx.status = 404;
    }
  }

  public static async updateUser(ctx: Context) {
    // ctx.body = `UpdateUser controller with ID = ${ctx.params.id}`;
    const userRepository = getManager().getRepository(User);
    await userRepository.update(+ctx.params.id, ctx.request.body);
    const updatedUser = await userRepository.findOne(+ctx.params.id);

    if (updatedUser) {
      ctx.status = 200;
      ctx.body = updatedUser;
    } else {
      ctx.status = 404;
    }
  }

  public static async deleteUser(ctx: Context) {
    // ctx.body = `DeleteUser controller with ID = ${ctx.params.id}`;
    const userRepository = getManager().getRepository(User);
    await userRepository.delete(+ctx.params.id);

    ctx.status = 204;
  }
}
```

`TypeORM` 中操作数据模型主要是通过 `Repository` 实现的，在 `Controller` 中，可以通过 `getManager().getRepository(Model)` 来获取到，之后 `Repository` 的查询 `API` 就与其他的库很类似了。

> 关于 Repository 所有的查询 API，请参考[这里](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/repository-api.md)的文档。

还可以通过 `ctx.request.body` 获取到了请求体的数据，这是我们在第一步就配置好的 `bodyParser` 中间件在 `Context` 对象中添加的。

然后修改 AuthController ，实现具体的注册逻辑。由于密码不能明文保存在数据库中，需要使用`hash`算法进行加密，这里我们使用曾经获得过密码加密大赛冠军的 [Argon2](https://www.argon2.com/) 算法。安装对应的 npm 包

`npm install argon2`

然后实现具体的 `register Controller`，修改 `src/controllers/auth.ts`

```js
import { Context } from 'koa';
import * as argon2 from 'argon2';
import { getManager } from 'typeorm';

import { User } from '../entity/user';

export default class AuthController {
  public static async login(ctx: Context) {
    ctx.body = 'Login controller';
  }

  public static async register(ctx: Context) {
    // ctx.body = 'Register controller';
    const userRepository = getManager().getRepository(User);

    const newUser = new User();
    newUser.name = ctx.request.body.name;
    newUser.email = ctx.request.body.email;
    newUser.password = await argon2.hash(ctx.request.body.password);

    // 保存到数据库
    const user = await userRepository.save(newUser);

    ctx.status = 201;
    ctx.body = user;
  }
}
```

使用`postman`调用`localhost:3000/auth/register`，并且传入对应的参数

在数据库中会看到记录了对应的数据

## JWT 鉴权

`JSON Web Token（JWT）`是一种流行的 `RESTful API` 鉴权方案。这里我们将手把手带你学会如何在 `Koa` 框架中使用 `JWT` 鉴权，可参考[这篇文章](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)进行学习。

首先安装相关的 npm 包

```bash
npm install koa-jwt jsonwebtoken
npm install @types/jsonwebtoken -D
```

生成密钥，实际生产环境需要利用非对称加密生成

```js
export const JWT_SECRET = "secret";
```

### 重新规划路由

有些路由我们希望只有已登录的用户才有权查看（受保护的路由），而另一些路由则是所有请求都可以访问（不受保护的路由）。在 `Koa` 的洋葱模型中，我们可以这样实现

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1294b1ce42a24120ab463e2993cbbda9~tplv-k3u1fbpfcp-watermark.image)

所有请求都可以直接访问未受保护的路由，但是受保护的路由就放在 `JWT` 中间件的后面（或者从洋葱模型的角度看是 “里面”），这样对于没有携带 `JWT Token` 的请求就直接返回，而不会继续传递下去

修改`src/routes.ts`

```js
import Router from "@koa/router";

import AuthController from "./controllers/auth";
import UserController from "./controllers/user";

const unprotectedRouter = new Router();

// auth 相关的路由
unprotectedRouter.post("/auth/login", AuthController.login);
unprotectedRouter.post("/auth/register", AuthController.register);

const protectedRouter = new Router();

// users 相关的路由
protectedRouter.get("/users", UserController.listUsers);
protectedRouter.get("/users/:id", UserController.showUserDetail);
protectedRouter.put("/users/:id", UserController.updateUser);
protectedRouter.delete("/users/:id", UserController.deleteUser);

export { protectedRouter, unprotectedRouter };
```

### 注册 JWT 中间件

修改`src/server.ts`

```js
// ...
import jwt from "koa-jwt";
import "reflect-metadata";

// import router from './routes';
import { protectedRouter, unprotectedRouter } from "./routes";
import { logger } from "./logger";
import { JWT_SECRET } from "./constants";

createConnection().then(() => {
  // ...

  // 响应用户请求
  // app.use(router.routes()).use(router.allowedMethods());
  // 无需 JWT Token 即可访问
  app.use(unprotectedRouter.routes()).use(unprotectedRouter.allowedMethods());

  // 注册 JWT 中间件
  // app.use(jwt({ secret: JWT_SECRET }).unless({ method: 'GET' }));
  // 需要 JWT Token 才可访问
  app.use(protectedRouter.routes()).use(protectedRouter.allowedMethods());

  // ...
});
// ...
```

> 在 JWT 中间件注册完毕后，如果用户请求携带了有效的 `Token`，后面的 `protectedRouter` 就可以通过 `ctx.state.user` 获取到 `Token` 的内容（更精确的说法是 `Payload`，负载，一般是用户的关键信息，例如 ID）了；反之，如果 `Token` 缺失或无效，那么 `JWT` 中间件会直接自动返回 `401` 错误。关于 `koa-jwt` 的更多使用细节，请参考其[文档](https://github.com/koajs/jwt)。

### 在 Login 中签发 JWT Token

我们需要提供一个 API 端口让用户可以获取到 `JWT Token`，最合适的当然是登录接口 `/auth/login`。打开 `src/controllers/auth.ts` ，在 `login` 控制器中实现签发 `JWT Token` 的逻辑，代码如下

```js
// ...
import jwt from 'jsonwebtoken';

// ...
import { JWT_SECRET } from '../constants';

export default class AuthController {
  public static async login(ctx: Context) {
    // ctx.body = 'Login controller';
    const userRepository = getManager().getRepository(User);

    const user = await userRepository
      .createQueryBuilder()
      .where({ name: ctx.request.body.name })
      .addSelect('User.password')
      .getOne();

    if (!user) {
      ctx.status = 401;
      ctx.body = { message: '用户名不存在' };
    } else if (await argon2.verify(user.password, ctx.request.body.password)) {
      ctx.status = 200;
      ctx.body = { token: jwt.sign({ id: user.id }, JWT_SECRET) };
    } else {
      ctx.status = 401;
      ctx.body = { message: '密码错误' };
    }
  }

  // ...
}
```

在 `login` 中，我们首先根据用户名（请求体中的 `name` 字段）查询对应的用户，如果该用户不存在，则直接返回 `401`；存在的话再通过 `argon2.verify` 来验证请求体中的明文密码 `password` 是否和数据库中存储的加密密码是否一致，如果一致则通过 `jwt.sign` 签发 `Token`，如果不一致则还是返回 `401`。

这里的 `Token` 负载就是标识用户 `ID` 的对象 `{ id: user.id }` ，这样后面鉴权成功后就可以通过 `ctx.user.id` 来获取用户 `ID`。

### 在 User 控制器中添加访问控制

`Token` 的中间件和签发都搞定之后，最后一步就是在合适的地方校验用户的 `Token`，确认其是否有足够的权限。最典型的场景便是，在更新或删除用户时，我们要确保是用户本人在操作。打开 `src/controllers/user.ts` ，代码如下：

```js
// ...

export default class UserController {
  // ...

  public static async updateUser(ctx: Context) {
    const userId = +ctx.params.id;

    if (userId !== +ctx.state.user.id) {
      ctx.status = 403;
      ctx.body = { message: '无权进行此操作' };
      return;
    }

    const userRepository = getManager().getRepository(User);
    // await userRepository.update(+ctx.params.id, ctx.request.body);
    // const updatedUser = await userRepository.findOne(+ctx.params.id);
    await userRepository.update(userId, ctx.request.body);
    const updatedUser = await userRepository.findOne(userId);

    // ...
  }

  public static async deleteUser(ctx: Context) {
    const userId = +ctx.params.id;

    if (userId !== +ctx.state.user.id) {
      ctx.status = 403;
      ctx.body = { message: '无权进行此操作' };
      return;
    }

    const userRepository = getManager().getRepository(User);
    // await userRepository.delete(+ctx.params.id);
    await userRepository.delete(userId);

    ctx.status = 204;
  }
}
```

两个 `Controller` 的鉴权逻辑基本相同，我们通过比较 `ctx.params.id` 和 `ctx.state.user.id` 是否相同，如果不相同则返回 `403 Forbidden` 错误，相同则继续执行相应的数据库操作。

## 实现自定义错误（异常）

首先，让我们来实现一些自定义的错误（或者异常，本文不作区分）类。创建 `src/exceptions.ts` ，代码如下：

```js
export class BaseException extends Error {
  // 状态码
  status: number;
  // 提示信息
  message: string;
}

export class NotFoundException extends BaseException {
  status = 404;

  constructor(msg?: string) {
    super();
    this.message = msg || "无此内容";
  }
}

export class UnauthorizedException extends BaseException {
  status = 401;

  constructor(msg?: string) {
    super();
    this.message = msg || "尚未登录";
  }
}

export class ForbiddenException extends BaseException {
  status = 403;

  constructor(msg?: string) {
    super();
    this.message = msg || "权限不足";
  }
}
```

这里的错误类型参考了 [Nest.js](https://docs.nestjs.cn/7/exceptionfilters) 的设计。出于学习目的，这里作了简化，并且只实现了我们需要用到的错误

### 在 Controller 中使用自定义错误

接着我们便可以在 `Controller` 中使用刚才的自定义错误了。打开 `src/controllers/auth.ts`

```js
// ...
import { UnauthorizedException } from '../exceptions';

export default class AuthController {
  public static async login(ctx: Context) {
    // ...

    if (!user) {
      // ctx.status = 401;
      // ctx.body = { message: '用户名不存在' };
      throw new UnauthorizedException('用户名不存在');
    } else if (await argon2.verify(user.password, ctx.request.body.password)) {
      ctx.status = 200;
      ctx.body = { token: jwt.sign({ id: user.id }, JWT_SECRET) };
    } else {
      // ctx.status = 401;
      // ctx.body = { message: '密码错误' };
      throw new UnauthorizedException('密码错误');
    }
  }

  // ...
}
```

可以看到，我们将直接手动设置状态码和响应体的代码改成了简单的错误抛出，代码清晰了很多。

> Koa 的 Context 对象提供了一个便捷方法 throw ，同样可以抛出异常，例如 ctx.throw(400, 'Bad request')。

同样地，修改 `UserController` 相关的逻辑。修改 `src/controllers/user.ts`

```js
// ...
import { NotFoundException, ForbiddenException } from '../exceptions';

export default class UserController {
  // ...

  public static async showUserDetail(ctx: Context) {
    const userRepository = getManager().getRepository(User);
    const user = await userRepository.findOne(+ctx.params.id);

    if (user) {
      ctx.status = 200;
      ctx.body = user;
    } else {
      // ctx.status = 404;
      throw new NotFoundException();
    }
  }

  public static async updateUser(ctx: Context) {
    const userId = +ctx.params.id;

    if (userId !== +ctx.state.user.id) {
      // ctx.status = 403;
      // ctx.body = { message: '无权进行此操作' };
      // return;
      throw new ForbiddenException();
    }

    // ...
  }
 // ...
  public static async deleteUser(ctx: Context) {
    const userId = +ctx.params.id;

    if (userId !== +ctx.state.user.id) {
      // ctx.status = 403;
      // ctx.body = { message: '无权进行此操作' };
      // return;
      throw new ForbiddenException();
    }

    // ...
  }
}
```

### 添加错误处理中间件

最后，我们需要添加错误处理中间件来捕获在 `Controller` 中抛出的错误。打开 `src/server.ts` ，实现错误处理中间件

```js
// ...

createConnection().then(() => {
  // ...

  // 注册中间件
  app.use(logger());
  app.use(cors());
  app.use(bodyParser());

  app.use(async (ctx, next) => {
    try {
      await next();
    } catch (err) {
      // 只返回 JSON 格式的响应
      ctx.status = err.status || 500;
      ctx.body = { message: err.message };
    }
  });

  // ...
});
// ...
```

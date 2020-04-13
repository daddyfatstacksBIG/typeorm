# 数据库连接

-   [连接](#connection)
-   [什么是`Connection`](#什么是`Connection`)
-   [创建新的连接](#创建新的连接)
-   [使用`ConnectionManager`](#使用`ConnectionManager`)
-   [使用连接](#使用连接)

## 什么是`Connection`

只有在建立连接后才能与数据库进行交互。 TypeORM 的`Connection`不会像看起来那样设
置单个数据库连接，而是设置连接池。如果你对数据库连接感兴趣，请参
阅`QueryRunner`文档。 `QueryRunner`的每个实例都是一个独立的数据库连接。一旦调
用`Connection`的`connect`方法，就建立连接池设置。如果使用`createConnection`函数
设置连接，则会自动调用`connect`方法。调用`close`时会断开连接（关闭池中的所有连接
）。通常情况下，你只能在应用程序启动时创建一次连接，并在完全使用数据库后关闭它。
实际上，如果要为站点构建后端，并且后端服务器始终保持运行,则不需要关闭连接。

## 创建新的连接

有多种方法可以创建连接。但是最简单和最常用的方法是使
用`createConnection`和`createConnections`函数。

`createConnection` 创建单个连接：

```typescript
import { createConnection, Connection } from "typeorm";

const connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
});
```

只使用`url`和`type`也可以进行连接。

```js
createConnection({
    type: "postgres",
    url: "postgres://test:test@localhost/test"
});
```

`createConnections` 创建多个连接:

```typescript
import { createConnections, Connection } from "typeorm";

const connections = await createConnections([
    {
        name: "default",
        type: "mysql",
        host: "localhost",
        port: 3306,
        username: "test",
        password: "test",
        database: "test"
    },
    {
        name: "test2-connection",
        type: "mysql",
        host: "localhost",
        port: 3306,
        username: "test",
        password: "test",
        database: "test2"
    }
]);
```

这两种方式都根据你传递的连接选项创建`Connection`，并调用`connect`方法。另外你也
可以在项目的根目录中创建一个`ormconfig.json`文件
，`createConnection`和`createConnections`将自动从此文件中读取连接选项。项目的根
目录与`node_modules`目录的级别相同。

```typescript
import { createConnection, createConnections, Connection } from "typeorm";

// createConnection将从ormconfig.json / ormconfig.js / ormconfig.yml / ormconfig.env / ormconfig.xml 文件或特殊环境变量中加载连接选项
const connection: Connection = await createConnection();

// 你可以指定要创建的连接的名称
// （如果省略名称，则将创建没有指定名称的连接）
const secondConnection: Connection = await createConnection("test2-connection");

// 如果调用createConnections而不是createConnection
// 它将初始化并返回ormconfig文件中定义的所有连接
const connections: Connection[] = await createConnections();
```

不同的连接必须具有不同的名称。默认情况下，如果未指定连接名称，则为`default`。通
常在你使用多个数据库或多个连接配置时才会使用多连接。

创建连接后，你可以使用`getConnection`函数从应用程序中的任何位置使用它：

```typescript
import { getConnection } from "typeorm";

// 可以在调用createConnection后使用并解析
const connection = getConnection();

// 如果你有多个连接，则可以按名称获取连接
const secondConnection = getConnection("test2-connection");
```

应避免额外创建 classes/services 来存储和管理连接。此功能已嵌入到 TypeORM 中 - �
需过度工程并创建无用的抽象。

## 使用`ConnectionManager`

你可以使用`ConnectionManager`类创建连接。例如：

```typescript
import { getConnectionManager, ConnectionManager, Connection } from "typeorm";

const connectionManager = getConnectionManager();
const connection = connectionManager.create({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
});
await connection.connect(); // 执行连接
```

这不是常规创建连接的方法，但它可能对某些用户有用。例如，想要创建连接并存储其实例
,同时控制何时建立实际"connection"。你还可以创建和维护自己的`ConnectionManager`：

```typescript
import { getConnectionManager, ConnectionManager, Connection } from "typeorm";

const connectionManager = new ConnectionManager();
const connection = connectionManager.create({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
});
await connection.connect(); // 执行连接
```

但请注意，使用该方式，你将无法再使用`getConnection()` - 你需要存储连接管理器实例
，并使用`connectionManager.get`来获取所需的连接。

通常情况下为避免应用程序中出现不必要的复杂情况，应尽量少使用此方法，除非你确实认
为需要时才使用`ConnectionManager`。

## 使用连接

设置连接后，可以使用`getConnection`函数在应用程序的任何位置使用它：

```typescript
import { getConnection } from "typeorm";
import { User } from "../entity/User";

export class UserController {
    @Get("/users")
    getAll() {
        return getConnection().manager.find(User);
    }
}
```

你也可以使用`ConnectionManager＃get`来获取连接，但在大多数情况下使
用`getConnection()`就足够了。

使用 Connection，你可以对实体执行数据库操作，尤其是使用连接
的`EntityManager`和`Repository`。有关它们的更多信息，请参
阅[Entity Manager 和 Repository](working-with-entity-manager.md) 文档。

但一般来说，你不要太多使用`Connection`。大多数情况下，你只需创建连接并使
用`getRepository()`和`getManager()`来访问连接的管理器和存储库，而无需直接使用连
接对象：

```typescript
import { getManager, getRepository } from "typeorm";
import { User } from "../entity/User";

export class UserController {
    @Get("/users")
    getAll() {
        return getManager().find(User);
    }

    @Get("/users/:id")
    getAll(@Param("id") userId: number) {
        return getRepository(User).findOne(userId);
    }
}
```

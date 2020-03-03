<div align="center">
  <a href="http://typeorm.io/">
    <img src="https://github.com/typeorm/typeorm/raw/master/resources/logo_big.png" width="492" height="228">
  </a>
  <br>
  <br>
	<a href="https://travis-ci.org/typeorm/typeorm">
		<img src="https://travis-ci.org/typeorm/typeorm.svg?branch=master">
	</a>
	<a href="https://badge.fury.io/js/typeorm">
		<img src="https://badge.fury.io/js/typeorm.svg">
	</a>
	<a href="https://david-dm.org/typeorm/typeorm">
		<img src="https://david-dm.org/typeorm/typeorm.svg">
	</a>
	<a href="https://gitter.im/typeorm/typeorm?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge">
		<img src="https://badges.gitter.im/typeorm/typeorm.svg">
	</a>
  <br>
  <br>
</div>

TypeORM 是一个 [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) 框
架，它可以运行在 NodeJS、Browser、Cordova、PhoneGap、Ionic、React Native、Expo
和 Electron 平台上，可以与 TypeScript 和 JavaScript (ES5,ES6,ES7,ES8)一起使用。
它的目标是始终支持最新的 JavaScript 特性并提供额外的特性以帮助你开发任何使用数据
库的（不管是只有几张表的小型应用还是拥有多数据库的大型企业应用）应用程序。

不同于现有的所有其他 JavaScript ORM 框架，TypeORM 支持
[Active Record](./docs/active-record-data-mapper.md#what-is-the-active-record-pattern)
和
[Data Mapper](./docs/active-record-data-mapper.md#what-is-the-data-mapper-pattern)
模式，这意味着你可以以最高效的方式编写高质量的、松耦合的、可扩展的、可维护的应用
程序。

TypeORM 参考了很多其他优秀 ORM 的实现, 比如
[Hibernate](http://hibernate.org/orm/),
[Doctrine](http://www.doctrine-project.org/) 和
[Entity Framework](https://www.asp.net/entity-framework)。

TypeORM 的一些特性:

-   同时支持
    [DataMapper](./docs/active-record-data-mapper.md#what-is-the-data-mapper-pattern)
    和
    [ActiveRecord](./docs/active-record-data-mapper.md#what-is-the-active-record-pattern)
    (随你选择)
-   实体和列
-   数据库特性列类型
-   实体管理
-   存储库和自定义存储库
-   清晰的对象关系模型
-   关联（关系）
-   贪婪和延迟关系
-   单向的，双向的和自引用的关系
-   支持多重继承模式
-   级联
-   索引
-   事务
-   迁移和自动迁移
-   连接�
-   主从复制
-   使用多个数据库连接
-   使用多个数据库类型
-   跨数据库和跨模式查询
-   优雅的语法，灵活而强大的 QueryBuilder
-   左联接和内联接
-   使用联查查询的适当分页
-   查询缓存
-   原始结果流
-   日志
-   监听者和订阅者（钩子）
-   支持闭包表模式
-   在模型或者分离的配置文件中声明模式
-   json / xml / yml / env 格式的连接配置
-   支持 MySQL / MariaDB / Postgres / SQLite / Microsoft SQL Server / Oracle /
    SAP Hana / sql.js
-   支持 MongoDB NoSQL 数据库
-   可在 NodeJS / 浏览器 / Ionic / Cordova / React Native / Expo / Electron 平台
    上使用
-   支持 TypeScript 和 JavaScript
-   生成高性能、灵活、清晰和可维护的代码
-   遵循所有可能的最佳实践
-   命令行工具

还有更多...

通过使用 `TypeORM` 你的 `models` 看起来如下:

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    age: number;
}
```

逻辑操作如下:

```typescript
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.age = 25;
await repository.save(user);

const allUsers = await repository.find();
const firstUser = await repository.findOne(1); // 根据id查找
const timber = await repository.findOne({
    firstName: "Timber",
    lastName: "Saw"
});

await repository.remove(timber);
```

或者，如果你更喜欢使用 `ActiveRecord` 模式，也可以这样用：

```typescript
import { Entity, PrimaryGeneratedColumn, Column, BaseEntity } from "typeorm";

@Entity()
export class User extends BaseEntity {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    age: number;
}
```

逻辑操作如下所示:

```typescript
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.age = 25;
await user.save();

const allUsers = await User.find();
const firstUser = await User.findOne(1);
const timber = await User.findOne({ firstName: "Timber", lastName: "Saw" });

await timber.remove();
```

# 入门

## 安装

1. 通过 `npm` 安装:

    `npm install typeorm --save`

2. 你还需要安装 `reflect-metadata`:

    `npm install reflect-metadata --save`

    并且需要在应用程序的全局位置导入（例如在`app.ts`中）

    `import "reflect-metadata";`

3. 你可能还需要安装 node typings(以此来使用 Node 的智能提示):

    `npm install @types/node --save`

4. 安装数据库驱动:

    - **MySQL** 或者 **MariaDB**

        `npm install mysql --save` (也可以安装 `mysql2`)

    - **PostgreSQL**

        `npm install pg --save`

    - **SQLite**

        `npm install sqlite3 --save`

    - **Microsoft SQL Server**

        `npm install mssql --save`

    - **sql.js**

        `npm install sql.js --save`

    - **Oracle**

        `npm install oracledb --save`

        根据你使用的数据库，仅安装其中*一个*即可。要使 Oracle 驱动程序正常工作，
        需要按照其[站点](https://github.com/oracle/node-oracledb)中的安装说明进
        行操作。

    - **MongoDB** (试验性)

        `npm install mongodb --save`

    - **NativeScript**, **react-native** 和 **Cordova**

        查看 [支持的平台](/supported-platforms.md)

    - **SAP Hana**

        ```
        npm config set @sap:registry https://npm.sap.com
        npm i @sap/hana-client
        ol
        ```

##### TypeScript 配置

此外，请确保你使用的 TypeScript 编译器版本是**3.3**或更高版本，并且已经在
`tsconfig.json` 中启用了以下设置:

```json
"emitDecoratorMetadata": true,
"experimentalDecorators": true,
```

除此之外，你可能还需要在编译器选项的 `lib` 中启用 `es6`，或者安装 `es6-shim` 的
`@types`。

## 快速开始

快速上手 TypeORM 的方法是使用其 CLI 命令生成启动项目。但是只有在 NodeJS 应用程序
中使用 TypeORM 时，此操作才有效。如果你使用的是其他平台，请查
看[分步指南](#分步指南)。

首先全局安装 TypeORM:

```
npm install typeorm -g
```

然后转到要创建新项目的目录并运行命令：

```
typeorm init --name MyProject --database mysql
```

其中 `name` 是项目的名称，`database` 是将使用的数据库。

数据库可以是以下值之一: `mysql`、 `mariadb`、 `postgres`、 `sqlite`、 `mssql`、
`oracle`、 `mongodb`、 `cordova`、 `react-native`、 `expo`、 `nativescript`.

此命令将在 `MyProject` 目录中生成一个包含以下文件的新项目:

```
MyProject
├── src              // TypeScript 代码
│   ├── entity       // 存储实体（数据库模型）的位置
│   │   └── User.ts  // 示例 entity
│   ├── migration    // 存储迁移的目录
│   └── index.ts     // 程序执行主文件
├── .gitignore       // gitignore文件
├── ormconfig.json   // ORM和数据库连接配置
├── package.json     // node module 依赖
├── README.md        // 简单的 readme 文件
└── tsconfig.json    // TypeScript 编译选项
```

> 你还可以在现有 node 项目上运行 `typeorm init`，但要注意，此操作可能会覆盖已有
> 的某些文件。

接下来安装项目依赖项：

```
cd MyProject
npm install
```

在安装过程中，编辑 `ormconfig.json` 文件并在其中编辑自己的数据库连接配置选项：

```json
{
    "type": "mysql",
    "host": "localhost",
    "port": 3306,
    "username": "test",
    "password": "test",
    "database": "test",
    "synchronize": true,
    "logging": false,
    "entities": ["src/entity/**/*.ts"],
    "migrations": ["src/migration/**/*.ts"],
    "subscribers": ["src/subscriber/**/*.ts"]
}
```

绝大多数情况下，你只需要配置 `host`, `username`, `password`, `database` 或者
`port` 即可。

完成配置并安装所有 node modules 后，即可运行应用程序：

```
npm start
```

至此你的应用程序应该成功运行并将新用户插入数据库。你可以继续使用此项目并集成所需
的其他模块并创建更多实体。

> 你可以通过运行 `typeorm init --name MyProject --database mysql --express` 来生
> 成一个更高级的 Express 项目

## 分步指南

你对 ORM 有何期待？期望它将为你创建数据库表，并且无需编写大量难以维护的 SQL 语句
来查找/插入/更新/删除数据。本指南将向你展示如何从头开始设置 TypeORM 并实现这些操
作。

### 创建一个模型

使用数据库从创建表开始。如何告诉 TypeORM 创建数据库表？答案是 - 通过模型。应用程
序中的模型即是数据库中的表。

举个例子, 你有一个 `Photo` 模型:

```typescript
export class Photo {
    id: number;
    name: string;
    description: string;
    filename: string;
    views: number;
}
```

并且希望将 photos 存储在数据库中。要在数据库中存储内容，首先需要一个数据库表，并
从模型中创建数据库表。但是并非所有模型，只有定义为*entities*的模型。

### 创建一个实体

*实体*是由 `@Entity` 装饰器装饰的模型。将为此类模型创建数据库表。你可以使用
TypeORM 处理各处的实体，可以使用它们 load/insert/update/remove 并执行其他操作。

让我们将 `Photo` 模型作为一个实体

```typescript
import { Entity } from "typeorm";

@Entity()
export class Photo {
    id: number;
    name: string;
    description: string;
    filename: string;
    views: number;
    isPublished: boolean;
}
```

现在，将为 `Photo` 实体创建一个数据库表，我们将能够在应用程序中的任何位置使用它
。我们已经创建了一个数据库表，但是没有指明哪个字段属于哪一列，下面让我们在数据库
表中创建列。

### 添加表列

要添加数据库列，你只需要将要生成的实体属性加上 `@Column` 装饰器。

```typescript
import { Entity, Column } from "typeorm";

@Entity()
export class Photo {
    @Column()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

现在 `id`, `name`, `description`, `filename`, `views` 和 `isPublished` 列将会被
添加到 `photo` 表中。数据库中的列类型是根据你使用的属性类型推断的，例如：
`number` 将被转换为 `integer`，`string` 将转换为 `varchar`，`boolean` 转换为
`bool` 等。但你也可以通过 `@Column` 装饰器中隐式指定列类型来使用数据库支持的任何
列类型。

我们已经生成了一个包含列的数据库表，但是别忘了，每个数据库表必须具有包含主键的列
。

### 创建主列

每个**必须**至少有一个主键列。这是必须的，你无法避免。要使列成为主键，你需要使用
`@PrimaryColumn` 装饰器。

```typescript
import { Entity, Column, PrimaryColumn } from "typeorm";

@Entity()
export class Photo {
    @PrimaryColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

### 创建自动生成的列

假设你希望 id 列自动生成（这称为 auto-increment/sequence/serial/generated
identity column）。为此你需要将`@PrimaryColumn` 装饰器更改为
`@PrimaryGeneratedColumn` 装饰器：

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

### 列数据类型

接下来，让我们修改数据类型。默认情况下，字符串被映射到一个 varchar(255) 类型（取
决于数据库类型）。数字被映射到一个类似 integer 类型（取决于数据库类型）。但是我
们不希望所有的列都是有限的 varchars 或 integer，让我们修改下代码以设置想要的数据
类型：

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm";

@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        length: 100
    })
    name: string;

    @Column("text")
    description: string;

    @Column()
    filename: string;

    @Column("double")
    views: number;

    @Column()
    isPublished: boolean;
}
```

列类型是特定于数据库的。你可以设置数据库支持的任何列类型。有关支持的列类型的更多
信息，请参见[此处](./docs/entities.md#column-types)。

### 创建数据库的连接

当实体被创建后，让我们创建一个`index.ts`（或`app.ts`，无论你怎么命名）文件，并配
置数据库连接：:

```typescript
import "reflect-metadata";
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "test",
    entities: [Photo],
    synchronize: true,
    logging: false
})
    .then(connection => {
        // 这里可以写实体操作相关的代码
    })
    .catch(error => console.log(error));
```

我们在此示例中使用 MySQL，你可以使用任何其他受支持的数据库。要使用其他数据库，只
需将选项中的 `type` 更改为希望使用的数据库类型
：`mysql`，`mariadb`，`postgres`，`sqlite`，`mssql`，`oracle`，`cordova`，`nativescript`，`react-native`，`expo`
或 `mongodb`。同时还要确保 `host`, `port`, `username`, `password` 和 `database`
正确设置。

我们将 Photo 实体添加到此连接的实体列表中，并且所有需要使用的实体都必须加进来。

设置 `synchronize` 可确保每次运行应用程序时实体都将与数据库同步。

### 加载目录中所有实体

之后当我们创建更多实体时，都需要一一将它们添加到配置中的实体中，但是这不是很方便
，所以我们可以设置加载整个目录，从中连接所有实体并使用：

```typescript
import { createConnection } from "typeorm";

createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "test",
    entities: [__dirname + "/entity/*.js"],
    synchronize: true
})
    .then(connection => {
        // 这里可以写实体操作相关的代码
    })
    .catch(error => console.log(error));
```

但要小心使用这种方法。如果使用的是 `ts-node`，则需要指定 `.ts` 文件的路径。如果
使用的是 `outDir`，那么需要在 `outDir` 目录中指定 `.js` 文件的路径。如果使用
`outDir`，当你删除或重命名实体时，请确保清除 `outDir` 目录并再次重新编译项目，�
为当你删除 `.ts` 源文件时，其编译的 `.js` 文件不会从输出目录中删除，并且 TypeORM
依然会从 `outDir` 中加载这些文件，从而导致异常。

### 启动应用

现在可以启动 `app.ts`，启动后可以发现数据库自动被初始化，并且 Photo 这个表也会创
建出来。

```bash
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(100) |                            |
| description | text         |                            |
| filename    | varchar(255) |                            |
| views       | int(11)      |                            |
| isPublished | boolean      |                            |
+-------------+--------------+----------------------------+
```

### 添加和插入 photo

现在创建一个新的 photo 存到数据库：

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/)
    .then(connection => {
        let photo = new Photo();
        photo.name = "Me and Bears";
        photo.description = "I am near polar bears";
        photo.filename = "photo-with-bears.jpg";
        photo.views = 1;
        photo.isPublished = true;

        return connection.manager.save(photo).then(photo => {
            console.log("Photo has been saved. Photo id is", photo.id);
        });
    })
    .catch(error => console.log(error));
```

保存实体后，将获得新生成的 ID。 `save` 方法返回传递给它的同一对象的实例，但并不
是对象的新副本，只是修改了它的"id"并返回。

### 使用 async/await 语法

我们可以使用 ES8(ES2017)的新特性，并使用 async/await 语法代替：

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/)
    .then(async connection => {
        let photo = new Photo();
        photo.name = "Me and Bears";
        photo.description = "I am near polar bears";
        photo.filename = "photo-with-bears.jpg";
        photo.views = 1;
        photo.isPublished = true;

        await connection.manager.save(photo);
        console.log("Photo has been saved");
    })
    .catch(error => console.log(error));
```

### 使用 Entity Manager

我们刚创建了一张新 photo 表并将其保存在数据库中。通过使用 `EntityManager` 你可以
操纵应用中的任何实体。

例如，加载已经保存的实体：

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/)
    .then(async connection => {
        /*...*/
        let savedPhotos = await connection.manager.find(Photo);
        console.log("All photos from the db: ", savedPhotos);
    })
    .catch(error => console.log(error));
```

`savedPhotos` 是一个 Photo 对象数组，其中包含从数据库加载的数据。

了解更多有关 [EntityManager](./docs/working-with-entity-manager.md) 的信息。

### 使用 Repositories

现在让我们重构之前的代码，并使用 `Repository` 替代 `EntityManager`。每个实体都有
自己的 repository，可以处理其实体的所有操作。当你经常处理实体时，Repositories 比
EntityManagers 更方便使用：

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/)
    .then(async connection => {
        let photo = new Photo();
        photo.name = "Me and Bears";
        photo.description = "I am near polar bears";
        photo.filename = "photo-with-bears.jpg";
        photo.views = 1;
        photo.isPublished = true;

        let photoRepository = connection.getRepository(Photo);

        await photoRepository.save(photo);
        console.log("Photo has been saved");

        let savedPhotos = await photoRepository.find();
        console.log("All photos from the db: ", savedPhotos);
    })
    .catch(error => console.log(error));
```

了解更多有关 [Repository](./docs/working-with-repository.md) 的信息。

### 从数据库加载

让我们使用 Repository 尝试更多的加载操作:

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/)
    .then(async connection => {
        /*...*/
        let allPhotos = await photoRepository.find();
        console.log("All photos from the db: ", allPhotos);

        let firstPhoto = await photoRepository.findOne(1);
        console.log("First photo from the db: ", firstPhoto);

        let meAndBearsPhoto = await photoRepository.findOne({
            name: "Me and Bears"
        });
        console.log("Me and Bears photo from the db: ", meAndBearsPhoto);

        let allViewedPhotos = await photoRepository.find({ views: 1 });
        console.log("All viewed photos: ", allViewedPhotos);

        let allPublishedPhotos = await photoRepository.find({
            isPublished: true
        });
        console.log("All published photos: ", allPublishedPhotos);

        let [allPhotos, photosCount] = await photoRepository.findAndCount();
        console.log("All photos: ", allPhotos);
        console.log("Photos count: ", photosCount);
    })
    .catch(error => console.log(error));
```

### 从数据库中更新

让我们从数据库加载出 photo，更新并保存到数据库：

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/)
    .then(async connection => {
        /*...*/
        let photoToUpdate = await photoRepository.findOne(1);
        photoToUpdate.name = "Me, my friends and polar bears";
        await photoRepository.save(photoToUpdate);
    })
    .catch(error => console.log(error));
```

这个 `id = 1` 的 photo 在数据库中就成功更新了。

### 从数据库中删除

让我们从数据库中删除 Photo:

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";

createConnection(/*...*/)
    .then(async connection => {
        /*...*/
        let photoToRemove = await photoRepository.findOne(1);
        await photoRepository.remove(photoToRemove);
    })
    .catch(error => console.log(error));
```

这个 `id = 1`的 photo 在数据库中被移除了。

### 创建一对一的关系

要与另一个类创建一对一的关系。先在 `PhotoMetadata.ts` 中创建一个新类。此
PhotoMetadata 类应包含 photo 的其他元信息：

```typescript
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    OneToOne,
    JoinColumn
} from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class PhotoMetadata {
    @PrimaryGeneratedColumn()
    id: number;

    @Column("int")
    height: number;

    @Column("int")
    width: number;

    @Column()
    orientation: string;

    @Column()
    compressed: boolean;

    @Column()
    comment: string;

    @OneToOne(type => Photo)
    @JoinColumn()
    photo: Photo;
}
```

这里我们使用了一个名为 `@OneToOne` 的新装饰器，它允许我们在两个实体之间创建一对
一的关系。 `type => Photo` 是一个函数，返回我们想要与之建立关系的实体的类。由于
特定于语言的关系，我们只能使用一个返回类的函数，而不是直接使用该类。同时也可以把
它写成 `()=> Photo`，但是 `type => Photo` 显得代码更有可读性。type 变量本身不包
含任何内容。

我们还添加了一个 `@JoinColumn` 装饰器，表明实体键的对应关系。关系可以是单向的或
双向的。但是只有一方可以拥有。在关系的所有者方需要使用 `@JoinColumn` 装饰器。

如果运行该应用程序，你将看到一个新生成的表，它将包含一个带有外键的列：

```bash
+-------------+--------------+----------------------------+
|                     photo_metadata                      |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| height      | int(11)      |                            |
| width       | int(11)      |                            |
| comment     | varchar(255) |                            |
| compressed  | boolean      |                            |
| orientation | varchar(255) |                            |
| photoId     | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

### 保存一对一的关系

现在让我们来创建一个 photo，它的元信息将它们互相连接起来。

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/)
    .then(async connection => {
        // 创建 photo
        let photo = new Photo();
        photo.name = "Me and Bears";
        photo.description = "I am near polar bears";
        photo.filename = "photo-with-bears.jpg";
        photo.isPublished = true;

        // 创建 photo metadata
        let metadata = new PhotoMetadata();
        metadata.height = 640;
        metadata.width = 480;
        metadata.compressed = true;
        metadata.comment = "cybershoot";
        metadata.orientation = "portait";
        metadata.photo = photo; // 联接两者

        // 获取实体 repositories
        let photoRepository = connection.getRepository(Photo);
        let metadataRepository = connection.getRepository(PhotoMetadata);

        // 先保存photo
        await photoRepository.save(photo);

        // 然后保存photo的metadata
        await metadataRepository.save(metadata);

        // 完成
        console.log(
            "Metadata is saved, and relation between metadata and photo is created in the database too"
        );
    })
    .catch(error => console.log(error));
```

### 反向关系

关系可以是单向的或双向的。目前 PhotoMetadata 和 Photo 之间的关系是单向的。关系的
所有者是 PhotoMetadata，而 Photo 对 PhotoMetadata 一无所知。这使得从 Photo 中访
问 PhotoMetadata 变得很复杂。要解决这个问题，我们应该在 PhotoMetadata 和 Photo
之间建立双向关系。让我们来修改一下实体：

```typescript
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    OneToOne,
    JoinColumn
} from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class PhotoMetadata {
    /* ... 其他列 */

    @OneToOne(
        type => Photo,
        photo => photo.metadata
    )
    @JoinColumn()
    photo: Photo;
}
```

```typescript
import { Entity, Column, PrimaryGeneratedColumn, OneToOne } from "typeorm";
import { PhotoMetadata } from "./PhotoMetadata";

@Entity()
export class Photo {
    /* ... 其他列 */

    @OneToOne(
        type => PhotoMetadata,
        photoMetadata => photoMetadata.photo
    )
    metadata: PhotoMetadata;
}
```

`photo => photo.metadata` 是用来指定反向关系的名称。Photo 类的元数据属性是在
Photo 类中存储 PhotoMetadata 的地方。你可以选择简单地将字符串传递给 `@OneToOne`
装饰器，而不是传递返回 photo 属性的函数，例如 `"metadata"`。这种函数类型的方法使
我们的重构更容易。

注意，我们应该仅在关系的一侧使用 `@JoinColumn` 装饰器。你把这个装饰者放在哪一方
将是这段关系的拥有方。关系的拥有方包含数据库中具有外键的列。

### 取出关系对象的数据

在一个查询中加载 photo 及 photo metadata 有两种方法。使用 `find *` 或使用
`QueryBuilder`。我们先使用 `find *` 方法。 `find *` 方法允许你使用
`FindOneOptions` / `FindManyOptions` 接口指定对象。

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/)
    .then(async connection => {
        /*...*/
        let photoRepository = connection.getRepository(Photo);
        let photos = await photoRepository.find({ relations: ["metadata"] });
    })
    .catch(error => console.log(error));
```

photos 包含来自数据库的 photos 数组，每个 photo 包含其 photo metadata。详细了解
本文档中的[Find 选项](./docs/find-options.md)。

使用 find 选项很简单，但是如果你需要更复杂的查询，则应该使用 `QueryBuilder`。
`QueryBuilder` 使用更优雅的方式执行更复杂的查询：

```typescript
import { createConnection } from "typeorm";
import { Photo } from "./entity/Photo";
import { PhotoMetadata } from "./entity/PhotoMetadata";

createConnection(/*...*/)
    .then(async connection => {
        /*...*/
        let photos = await connection
            .getRepository(Photo)
            .createQueryBuilder("photo")
            .innerJoinAndSelect("photo.metadata", "metadata")
            .getMany();
    })
    .catch(error => console.log(error));
```

`QueryBuilder` 允许你创建和执行几乎任何复杂性的 SQL 查询。使用 `QueryBuilder` 时
，请考虑创建 SQL 查询。在此示例中，"photo"和"metadata"是应用于所选 photos 的别名
。你可以使用别名来访问所选数据的列和属性。

### 使用 cascades 自动保存相关对象

我们可以在关系中设置 `cascade` 选项，这时就可以在保存其他对象的同时保存相关对象
。让我们更改一下的 photo 的 `@OneToOne` 装饰器：

```typescript
export class Photo {
    /// ... 其他列

    @OneToOne(
        type => PhotoMetadata,
        metadata => metadata.photo,
        {
            cascade: true
        }
    )
    metadata: PhotoMetadata;
}
```

使用 `cascade` 允许就不需要边存 photo 边存元数据对象。我们可以简单地保存一个
photo 对象，由于使用了 cascade，metadata 也将自动保存。

```typescript
createConnection(options)
    .then(async connection => {
        // 创建 photo 对象
        let photo = new Photo();
        photo.name = "Me and Bears";
        photo.description = "I am near polar bears";
        photo.filename = "photo-with-bears.jpg";
        photo.isPublished = true;

        // 创建 photo metadata 对象
        let metadata = new PhotoMetadata();
        metadata.height = 640;
        metadata.width = 480;
        metadata.compressed = true;
        metadata.comment = "cybershoot";
        metadata.orientation = "portait";

        photo.metadata = metadata; // this way we connect them

        // 获取 repository
        let photoRepository = connection.getRepository(Photo);

        // 保存photo的同时保存metadata
        await photoRepository.save(photo);

        console.log("Photo is saved, photo metadata is saved too.");
    })
    .catch(error => console.log(error));
```

### 创建多对一/一对多关系

让我们创建一个多对一/一对多的关系。假设一个 photo 有一个 author，每个 author 都
可以有多个 photos。首先让我们创建一个`Author`类：

```typescript
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    OneToMany,
    JoinColumn
} from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class Author {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(
        type => Photo,
        photo => photo.author
    ) // 注意：我们将在下面的Photo类中创建author属性
    photos: Photo[];
}
```

`Author` 包含反向关系。 `OneToMany` 总是反向的, 并且总是与 `ManyToOne`一起出现。

现在让我们将关系的所有者方添加到 Photo 实体中：

```typescript
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from "typeorm";
import { PhotoMetadata } from "./PhotoMetadata";
import { Author } from "./Author";

@Entity()
export class Photo {
    /* ... other columns */

    @ManyToOne(
        type => Author,
        author => author.photos
    )
    author: Author;
}
```

在多对一/一对多的关系中，拥有方总是多对一的。这意味着使用`@ManyToOne`的类将存储
相关对象的 id。运行应用程序后，ORM 将创建`author`表：

```bash
+-------------+--------------+----------------------------+
|                          author                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+
```

它还将修改`photo`表，添加新的`author`列并为其创建外键：

```bash
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
| description | varchar(255) |                            |
| filename    | varchar(255) |                            |
| isPublished | boolean      |                            |
| authorId    | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

### 创建多对多关系

假设一个 photo 可以放在多个 albums 中，每个 albums 可以包含多个 photo。让我们创
建一个`Album`类：

```typescript
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable
} from "typeorm";

@Entity()
export class Album {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(
        type => Photo,
        photo => photo.albums
    )
    @JoinTable()
    photos: Photo[];
}
```

`@JoinTable`需要指定这是关系的所有者方。

现在添加反向关系到`Photo`类：

```typescript
export class Photo {
    /// ... 其他列

    @ManyToMany(
        type => Album,
        album => album.photos
    )
    albums: Album[];
}
```

运行后，ORM 将创建**album_photos_photo_albums**\_联结表。

```bash
+-------------+--------------+----------------------------+
|                album_photos_photo_albums                |
+-------------+--------------+----------------------------+
| album_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
| photo_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```

记得在 ORM 中使用 ConnectionOptions 注册`Album`类：

```typescript
const options: ConnectionOptions = {
    // ... 其他选项
    entities: [Photo, PhotoMetadata, Author, Album]
};
```

现在让我们将 albums 和 photos 插入我们的数据库:

```typescript
let connection = await createConnection(options);

// 创建一些 albums
let album1 = new Album();
album1.name = "Bears";
await connection.manager.save(album1);

let album2 = new Album();
album2.name = "Me";
await connection.manager.save(album2);

// 创建一些 photos
let photo = new Photo();
photo.name = "Me and Bears";
photo.description = "I am near polar bears";
photo.filename = "photo-with-bears.jpg";
photo.albums = [album1, album2];
await connection.manager.save(photo);

// 现在我们的`photo`被保存了，并且'albums`被附加到它上面
// 然后加载它们
const loadedPhoto = await connection
    .getRepository(Photo)
    .findOne(1, { relations: ["albums"] });
```

`loadedPhoto` 如下所示:

```typescript
{
    id: 1,
    name: "Me and Bears",
    description: "I am near polar bears",
    filename: "photo-with-bears.jpg",
    albums: [{
        id: 1,
        name: "Bears"
    }, {
        id: 2,
        name: "Me"
    }]
}
```

### 使用 QueryBuilder

你可以使用 QueryBuilder 构建几乎任何复杂性的 SQL 查询。例如，可以这样做：

```typescript
let photos = await connection
    .getRepository(Photo)
    .createQueryBuilder("photo") // 第一个参数是别名。即photos。 该参数必须指定。
    .innerJoinAndSelect("photo.metadata", "metadata")
    .leftJoinAndSelect("photo.albums", "album")
    .where("photo.isPublished = true")
    .andWhere("(photo.name = :photoName OR photo.name = :bearName)")
    .orderBy("photo.id", "DESC")
    .skip(5)
    .take(10)
    .setParameters({ photoName: "My", bearName: "Mishka" })
    .getMany();
```

此查询选择所有 published 的 name 等于"My"或"Mishka"的 photos。它将从结果中的第 5
个（分页偏移）开始，并且仅选择 10 个结果（分页限制）。得到的结果将按 ID 降序排序
。photo 的 albums 将被 left-joined，其元数据将被 inner joined。

由于 QueryBuilder 的自由度更高，因此在项目中可能会大量的使用它。更多关于
QueryBuilder 的信息，[可查看](./docs/select-query-builder.md)。

## 示例

查看[示例](https://github.com/typeorm/typeorm/tree/master/sample)用法。

下面这些 repositories 可以帮助你快速开始：

-   [Example how to use TypeORM with TypeScript](https://github.com/typeorm/typescript-example)
-   [Example how to use TypeORM with JavaScript](https://github.com/typeorm/javascript-example)
-   [Example how to use TypeORM with JavaScript and Babel](https://github.com/typeorm/babel-example)
-   [Example how to use TypeORM with TypeScript and SystemJS in Browser](https://github.com/typeorm/browser-example)
-   [Example how to use Express and TypeORM](https://github.com/typeorm/typescript-express-example)
-   [Example how to use Koa and TypeORM](https://github.com/typeorm/typescript-koa-example)
-   [Example how to use TypeORM with MongoDB](https://github.com/typeorm/mongo-typescript-example)
-   [Example how to use TypeORM in a Cordova/PhoneGap app](https://github.com/typeorm/cordova-example)
-   [Example how to use TypeORM with an Ionic app](https://github.com/typeorm/ionic-example)
-   [Example how to use TypeORM with React Native](https://github.com/typeorm/react-native-example)
-   [Example how to use TypeORM with Electron using JavaScript](https://github.com/typeorm/electron-javascript-example)
-   [Example how to use TypeORM with Electron using TypeScript](https://github.com/typeorm/electron-typescript-example)

## 扩展

这几个扩展可以简化 TypeORM 的使用，并将其与其他模块集成：

-   [TypeORM + GraphQL framework](http://vesper-framework.com)
-   [TypeORM integration](https://github.com/typeorm/typeorm-typedi-extensions)
    with [TypeDI](https://github.com/pleerock/typedi)
-   [TypeORM integration](https://github.com/typeorm/typeorm-routing-controllers-extensions)
    with [routing-controllers](https://github.com/pleerock/routing-controllers)
-   从现有数据库生成模型 -
    [typeorm-model-generator](https://github.com/Kononnable/typeorm-model-generator)

## 贡献

了解如何贡
献[这里](https://github.com/typeorm/typeorm/blob/master/CONTRIBUTING.md)以及如何
设置开发环
境[这里](https://github.com/typeorm/typeorm/blob/master/DEVELOPER.md)。

感谢所有贡献者：

<a href="https://github.com/typeorm/typeorm/graphs/contributors"><img src="https://opencollective.com/typeorm/contributors.svg?width=890&showBtn=false" /></a>

## 赞助商

开源既困难又耗时。 如果你想投资 TypeORM 的未来，你可以成为赞助商，让我们的核心团
队花更多时间在 TypeORM 的改进和新功能上
。[成为赞助商](https://opencollective.com/typeorm)

<a href="https://opencollective.com/typeorm" target="_blank"><img src="https://opencollective.com/typeorm/tiers/sponsor.svg?width=890"></a>

## 金牌赞助商

成为金牌赞助商，并从我们的核心贡献者那里获得高级技术支持。
[成为金牌赞助商](https://opencollective.com/typeorm)

<a href="https://opencollective.com/typeorm" target="_blank"><img src="https://opencollective.com/typeorm/tiers/gold-sponsor.svg?width=890"></a>

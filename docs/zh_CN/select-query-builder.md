# 使用 Query Builder 查询

-   [什么是`QueryBuilder`](#什么是`QueryBuilder`)
-   [如何创建和使用`QueryBuilder`](#如何创建和使用`QueryBuilder`)
-   [使用`QueryBuilder`获取值](#使用`QueryBuilder`获取值)
-   [什么是别名？](#什么是别名？)
-   [使用参数来转义数据](#使用参数来转义数据)
-   [添加`WHERE`表达式](#添加`WHERE`表达式)
-   [添加`HAVING`表达式](#添加`HAVING`表达式)
-   [添加`ORDER BY`表达式](#添加`ORDER-BY`表达式)
-   [添加`GROUP BY`表达式](#添加`GROUP-BY`表达式)
-   [添加`LIMIT`表达式](#添加`LIMIT`表达式)
-   [添加`OFFSET`表达式](#添加`OFFSET`表达式)
-   [联查](#联查)
-   [内联和左联](#内联和左联)
-   [不使用条件的联查](#不使用条件的联查)
-   [联查任何实体或表](#联查任何实体或表)
-   [联查和映射功能](#联查和映射功能)
-   [获取生成的 sql 查询语句](#获取生成的sql查询语句)
-   [获得原始结果](#获得原始结果)
-   [流数据](#流数据)
-   [分页](#分页)
-   [加锁](#加锁)
-   [查询部分字段](#查询部分字段)
-   [使用子查询](#使用子查询)
-   [隐藏列](#隐藏列)

## 什么是`QueryBuilder`

`QueryBuilder`是 TypeORM 最强大的功能之一 ，它允许你使用优雅便捷的语法构建 SQL
查询，执行并获得自动转换的实体。

`QueryBuilder`的简单示例:

```typescript
const firstUser = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne();
```

它将生成以下 SQL 查询：

```sql
SELECT
    user.id as userId,
    user.firstName as userFirstName,
    user.lastName as userLastName
FROM users user
WHERE user.id = 1
```

然后返回一个 `User` 实例:

```
User {
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}
```

## 如何创建和使用`QueryBuilder`

有几种方法可以创建`Query Builder`：

-   使用 connection:

    ```typescript
    import { getConnection } from "typeorm";

    const user = await getConnection()
        .createQueryBuilder()
        .select("user")
        .from(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

-   使用 entity manager:

    ```typescript
    import { getManager } from "typeorm";

    const user = await getManager()
        .createQueryBuilder(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

-   使用 repository:

    ```typescript
    import { getRepository } from "typeorm";

    const user = await getRepository(User)
        .createQueryBuilder("user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

有 5 种不同的`QueryBuilder`类型可用：

-   `SelectQueryBuilder` - 用于构建和执行`SELECT`查询。 例如：

    ```typescript
    import { getConnection } from "typeorm";

    const user = await getConnection()
        .createQueryBuilder()
        .select("user")
        .from(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

-   `InsertQueryBuilder` - 用于构建和执行`INSERT`查询。 例如：

    ```typescript
    import { getConnection } from "typeorm";

    await getConnection()
        .createQueryBuilder()
        .insert()
        .into(User)
        .values([
            { firstName: "Timber", lastName: "Saw" },
            { firstName: "Phantom", lastName: "Lancer" }
        ])
        .execute();
    ```

-   `UpdateQueryBuilder` - 用于构建和执行`UPDATE`查询。 例如：

    ```typescript
    import { getConnection } from "typeorm";

    await getConnection()
        .createQueryBuilder()
        .update(User)
        .set({ firstName: "Timber", lastName: "Saw" })
        .where("id = :id", { id: 1 })
        .execute();
    ```

-   `DeleteQueryBuilder` - 用于构建和执行`DELETE`查询。 例如：

    ```typescript
    import { getConnection } from "typeorm";

    await getConnection()
        .createQueryBuilder()
        .delete()
        .from(User)
        .where("id = :id", { id: 1 })
        .execute();
    ```

-   `RelationQueryBuilder` - 用于构建和执行特定于关系的操作[TBD]。

你可以在其中切换任何不同类型的查询构建器，一旦执行，则将获得一个新的查询构建器实
例（与所有其他方法不同）。

## 使用`QueryBuilder`获取值

要从数据库中获取单个结果，例如通过 id 或 name 获取用户，必须使用`getOne`：

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

使用查询构建器查询可以获得两种类型的结果：**entities** 或 **raw results**。大多
数情况下，你只需要从数据库中选择真实实体，例如 users。为此，你可以使
用`getOne`和`getMany`。但有时你需要选择一些特定的数据，比方说所有*sum of all
user photos*。此数据不是实体，它称为原始数据。要获取原始数据，请使
用`getRawOne`和`getRawMany`。例如：

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

## 什么是别名？

我们使用`createQueryBuilder（"user"）`。 但什么是"user"？它只是一个常规的 SQL 别
名。我们在任何地方都使用别名，除非我们处理选定的数据。

`createQueryBuilder("user")` 相当于：

```typescript
createQueryBuilder()
    .select("user")
    .from(User, "user");
```

这会生成以下 sql 查询：

```sql
SELECT ... FROM users user
```

在这个 SQL 查询中，`users`是表名，`user`是我们分配给该表的别名。

稍后我们使用此别名来访问表：

```typescript
createQueryBuilder()
    .select("user")
    .from(User, "user")
    .where("user.name = :name", { name: "Timber" });
```

以上代码会生成如下 SQL 语句：

```sql
SELECT ... FROM users user WHERE user.name = 'Timber'
```

看到了吧，我们使用了在创建查询构建器时分配的`user`别名来使用 users 表。

一个查询构建器不限于一个别名，它们可以有多个别名。每个选择都可以有自己的别名，�
可以选择多个有自己别名的表，你可以使用自己的别名连接多个表。你也可以使用这些别名
来访问选择的表（或正在选择的数据）。

## 使用参数来转义数据

我们使用了`where("user.name = :name", { name: "Timber" })`. `{name：“Timber”}`代
表什么？ 这是我们用来阻止 SQL 注入的参数。我们可以写
：`where（"user.name ='"+ name +"'）`，但是这不安全，因为有可能被 SQL 注入。安全
的方法是使用这种特殊的语法：`where（"user.name =name"，{name:"Timber"}）`，其
中`name`是参数名，值在对象中指定： `{name:"Timber"}`。

```typescript
.where("user.name = :name", { name: "Timber" })
```

是下面的简写：

```typescript
.where("user.name = :name")
.setParameter("name", "Timber")
```

注意：不要在查询构建器中为不同的值使用相同的参数名称。如果多次设置则后值将会把前
面的覆盖。

还可以提供一组值，并使用特殊的扩展语法将它们转换为 SQL 语句中的值列表：

```typescript
.where("user.name IN (:...names)", { names: [ "Timber", "Cristal", "Lina" ] })
```

该语句将生成：

```sql
WHERE user.name IN ('Timber', 'Cristal', 'Lina')
```

## 添加`WHERE`表达式

添加 `WHERE` 表达式就像：

```typescript
createQueryBuilder("user").where("user.name = :name", { name: "Timber" });
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user WHERE user.name = 'Timber'
```

你可以将 `AND` 添加到现有的 `WHERE` 表达式中：

```typescript
createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .andWhere("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user WHERE user.firstName = 'Timber' AND user.lastName = 'Saw'
```

你也可以添加 `OR` 添加到现有的 `WHERE` 表达式中：

```typescript
createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .orWhere("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 SQL 语句：

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

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user WHERE user.registered = true AND (user.firstName = 'Timber' OR user.lastName = 'Saw')
```

你可以根据需要组合尽可能多的`AND`和`OR`表达式。如果你多次使用`.where`，你将覆盖
所有以前的`WHERE`表达式。注意：小心`orWhere` - 如果你使用带有`AND`和`OR`表达式的
复杂表达式，请记住他们将无限制的叠加。有时你只需要创建一个 where 字符串，避免使
用`orWhere`。

## 添加`HAVING`表达式

添加`HAVING`表达式很简单：

```typescript
createQueryBuilder("user").having("user.name = :name", { name: "Timber" });
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user HAVING user.name = 'Timber'
```

你可以添加 `AND` 到已经存在的 `HAVING` 表达式中：

```typescript
createQueryBuilder("user")
    .having("user.firstName = :firstName", { firstName: "Timber" })
    .andHaving("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user HAVING user.firstName = 'Timber' AND user.lastName = 'Saw'
```

你可以添加 `OR` 到已经存在的 `HAVING` 表达式中：

```typescript
createQueryBuilder("user")
    .having("user.firstName = :firstName", { firstName: "Timber" })
    .orHaving("user.lastName = :lastName", { lastName: "Saw" });
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user HAVING user.firstName = 'Timber' OR user.lastName = 'Saw'
```

你可以根据需要组合尽可能多的`AND`和`OR`表达式。如果使用多个`.having`，后面的将覆
盖所有之前的`HAVING`表达式。

## 添加`ORDER BY`表达式

添加 `ORDER BY` 很简单：

```typescript
createQueryBuilder("user").orderBy("user.id");
```

将会生成一下 SQL 语句：

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

还可以使用排序字段作为一个 map：

```typescript
createQueryBuilder("user").orderBy({
    "user.name": "ASC",
    "user.id": "DESC"
});
```

如果你使用了多个`.orderBy`，后面的将覆盖所有之前的`ORDER BY`表达式。

## 添加`GROUP BY`表达式

添加 `GROUP BY` 表达式很简单：

```typescript
createQueryBuilder("user").groupBy("user.id");
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user GROUP BY user.id
```

如果要使用更多 group-by, 则可以使用 c`addGroupBy`:

```typescript
createQueryBuilder("user")
    .groupBy("user.name")
    .addGroupBy("user.id");
```

如果使用了多个`.groupBy` ，则后面的将会覆盖之前所有的 `ORDER BY` 表达式。

## 添加`LIMIT`表达式

添加 `LIMIT` 表达式很简单：

```typescript
createQueryBuilder("user").limit(10);
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user LIMIT 10
```

生成的 SQL 查询取决于数据库的类型（SQL，mySQL，Postgres 等）。注意：如果你使用带
有连接或子查询的复杂查询，LIMIT 可能无法正常工作。如果使用分页，建议使用`take`代
替。

## 添加`OFFSET`表达式

添加 SQL`OFFSET`表达式很简单：

```typescript
createQueryBuilder("user").offset(10);
```

将会生成以下 SQL 语句：

```sql
SELECT ... FROM users user OFFSET 10
```

生成的 SQL 查询取决于数据库的类型（SQL，mySQL，Postgres 等）。注意：如果你使用带
有连接或子查询的复杂查询，OFFSET 可能无法正常工作。如果使用分页，建议使
用`skip`代替。

## 联查

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

    @OneToMany(
        type => Photo,
        photo => photo.user
    )
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

    @ManyToOne(
        type => User,
        user => user.photos
    )
    user: User;
}
```

现在让我们假设你要用用户"Timber"加载他所有的 photos：

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

你可以看到`leftJoinAndSelect`自动加载了所有 Timber 的 photos。第一个参数是你要�
载的关系，第二个参数是你为此关系的表分配的别名。你可以在查询构建器中的任何位置使
用此别名。例如，让我们获得所有未删除的 Timber 的 photos。

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .where("user.name = :name", { name: "Timber" })
    .andWhere("photo.isRemoved = :isRemoved", { isRemoved: false })
    .getOne();
```

将会生成以下 SQL 查询：

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id
    WHERE user.name = 'Timber' AND photo.isRemoved = FALSE
```

你还可以向连接表达式添加条件，而不是使用"where"：

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", {
        isRemoved: false
    })
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

这将生成以下 sql 查询：

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'
```

## 内联和左联

如果你想使用`INNER JOIN`而不是`LEFT JOIN`，只需使用`innerJoinAndSelect`：

```typescript
const user = await createQueryBuilder("user")
    .innerJoinAndSelect(
        "user.photos",
        "photo",
        "photo.isRemoved = :isRemoved",
        { isRemoved: false }
    )
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

This will generate:

```sql
SELECT user.*, photo.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'
```

`LEFT JOIN`和`INNER JOIN`之间的区别在于，如果没有任何 photos，`INNER JOIN`将不会
返回 user。即使没有 photos，`LEFT JOIN`也会返回 user。要了解有关不同连接类型的更
多信息，请参阅
[SQL 文档](https://msdn.microsoft.com/en-us/library/zt8wzxy4.aspx).

## 不使用条件的联查

你可以在不使用条件的情况下联查数据。要做到这一点，使用`leftJoin`或`innerJoin`：

```typescript
const user = await createQueryBuilder("user")
    .innerJoin("user.photos", "photo")
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

将会生成如下 SQL 语句：

```sql
SELECT user.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id
    WHERE user.name = 'Timber'
```

这将会返回 Timber 如果他有 photos，但是并不会返回他的 photos。

## 联查任何实体或表

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

## 联查和映射功能

将`profilePhoto`添加到`User`实体，你可以使用`QueryBuilder`将任何数据映射到该属性
：

```typescript
export class User {
    /// ...
    profilePhoto: Photo;
}
```

```typescript
const user = await createQueryBuilder("user")
    .leftJoinAndMapOne(
        "user.profilePhoto",
        "user.photos",
        "photo",
        "photo.isForProfile = TRUE"
    )
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

这将加载 Timber 的个人资料照片并将其设置为`user.profilePhoto`。如果要加载并映射
单个实体，请使用`leftJoinAndMapOne`。如果要加载和映射多个实体，请使
用`leftJoinAndMapMany`。

## 获取生成的 sql 查询语句

有时你可能想要获取`QueryBuilder`生成的 SQL 查询。为此，请使用`getSql`：

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

此查询将返回 users 并将使用的 sql 语句打印到控制台。

## 获得原始结果

使用选择查询构建器可以获得两种类型的结果：**entities** 和 **raw results**。大多
数情况下，你只需要从数据库中选择真实实体，例如 users。为此，你可以使
用`getOne`和`getMany`。但是，有时需要选择特定数据，例如 _sum of all user
photos_。这些数据不是实体，它被称为原始数据。要获取原始数据，请使
用`getRawOne`和`getRawMany`。例如：

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

## 流数据

你可以使用`stream`来返回流。 Streaming 返回原始数据，必须手动处理实体转换：

```typescript
const stream = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .stream();
```

## 使用分页

大多数情况下，在开发应用程序时，你可能需要分页功能。如果你的应用程序中有分页
，page slider 或无限滚动组件，则使用此选项。

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .take(10)
    .getMany();
```

将会返回前 10 个 user 的 photos。

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(10)
    .getMany();
```

将返回除了前 10 个 user 以外的所有 user 的 photos。

你可以组合这些方法：

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(5)
    .take(10)
    .getMany();
```

这将跳过前 5 个 users，并获取他们之后的 10 个 user。

`take`和`skip`可能看起来像我们正在使用`limit`和`offset`，但它们不是。一旦你有更
复杂的连接或子查询查询，`limit`和`offset`可能无法正常工作。使用`take`和`skip`可
以防止这些问题。

## 加锁

QueryBuilder 支持 optimistic 和 pessimistic 锁定。要使用 pessimistic 读锁定，请
使用以下方式：

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("pessimistic_read")
    .getMany();
```

要使用 pessimistic 写锁定，请使用以下方式：

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("pessimistic_write")
    .getMany();
```

要使用 optimistic 读锁定，请使用以下方式：

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("optimistic", existUser.version)
    .getMany();
```

要使用 dirty 读锁定，请使用以下方式：

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .setLock("dirty_read")
    .getMany();
```

Optimistic 锁定与`@Version`和`@UpdatedDate`装饰器一起使用。

## 查询部分字段

如果只想选择实体的某些属性，可以使用以下语法：

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .select(["user.id", "user.name"])
    .getMany();
```

这只会选择`User`的`id`和`name`。

## 使用子查询

你可以轻松创建子查询。 `FROM`，`WHERE`和`JOIN`表达式都支持子查询。例如：

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

或者，你可以创建单独的查询构建器并使用其生成的 SQL：

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

如果想添加一个子查询做为"second from"，请使用`addFrom`。

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

## 隐藏列

如果要查询的模型具有"select：false"的列，则必须使用`addSelect`函数来从列中检索信
息。

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

使用标准的`find`或查询，你将不会接收到模型的`password`属性。 但是，如果执行以下
操作：

```typescript
const users = await connection
    .getRepository(User)
    .createQueryBuilder()
    .select("user.id", "id")
    .addSelect("user.password")
    .getMany();
```

你将在查询中获得属性`password`。

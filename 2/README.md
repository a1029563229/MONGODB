# 通过 Javascript shell 操作 MongoDB

## 深入 MongoDB shell
- 进入 `mongodb` 控制台
```bash
docker exec -it practice-mongodb mongo admin
```

- 切换数据库
```bash
use tutorial
```

其实创建数据库不是必须的。只有在第一次插入数据库和集合时才会创建。这个行为符合 MongoDB 动态操作数据的模式。正如文档的数据结构不需要提前定义一样，单个的数据库和集合也可以在运行时创建。

### 插入和查询
- 插入数据
```js
db.users.insert({ username: "smith" });
db.users.insert({ username: "dove" });
```

- 查询数据
```js
db.users.find();
// { "_id" : ObjectId("5d8177ee843491f5388f0cf0"), "username" : "smith" }
// { "_id" : ObjectId("5d81798d843491f5388f0cf1"), "username" : "dove" }
```
每个 MongoDB 文档都需要一个 _id，我们可以把 _id 字段作为文档的主键，该值是唯一的。

- 查询数据数量
```js
db.users.count();
```

- 条件查询
```js
// 简单查询
db.users.find({ username: "smith" });
// db.users.find() == db.users.find({})

// 多条件查询
db.users.find({ "_id" : ObjectId("5d8177ee843491f5388f0cf0"), "username" : "smith" });

// AND 查询
db.users.find({ $and: [
  { "_id" : ObjectId("5d8177ee843491f5388f0cf0") },
  { "username" : "smith" }
] });

// OR 查询
db.users.find({ $or: [
  { username: "smith" },
  { username: "dove" }
] });
```

### 更新文档
所有的更新至少需要 2 个参数，第一个指定要更新的文档，第二个定义要如何修改此文档。

- 局部更新
```js
// 找到一个用户名为 smith 的文档，然后把 country 属性设置为 Canada
db.users.update({ username: "smith" }, { $set: { country: "Canada" } });
```

- 替换更新
```js
// 用户名为 smith 的文档的字段除 _id 外都被删除，同时 sex 字段被插入
db.users.update({ username: "smith" }, { sex: "男" });
```

- 删除单个字段
```js
db.users.update({ sex: "男" }, { $unset: { sex: 1 } });
```

- 更新复杂数据
```js
db.users.update({ username: "smith" }, {
  $set: {
    favorites: {
      cities: [ "Chicago", "Cheyenne" ],
      movies: [ "Casablanca", "For a Few Dollars More", "The String" ]
    }
  }
});

db.users.update({ username: "dove" }, {
  $set: {
    favorites: {
      movies: [ "Casablanca", "Rocky" ]
    }
  }
});
```

- 格式化查询
```js
db.users.find().pretty();
```

- 深度条件查询
```js
db.users.find({ "favorites.movies": "Casablanca" }).pretty();
```

- 高级更新
```js
db.users.update({ "favorites.movies": "Casablanca" }, {
  $addToSet: {
    "favorites.movies": "The Maltese Falcon"
  }
}, 
false,
true);
// 第一个参数是查询条件
// 第二个参数是使用 $addToSet 添加 The Maltese Falcon 到列表中
// 第三个参数 false，控制是否允许 upsert。这个命令告诉更新操作，当一个文档不存在的时候是否插入它，这取决于更新操作是操作符更新还是替换更新；
// 第四个参数 true，表示是否是多个更新。默认情况下，MongoDB 更新只针对第一个匹配文档。
```

### 删除数据
- 删除所有数据
```js
db.foo.remove();
```

- 删除集合中的某个文档
```js
db.users.remove({ "favorites.cities": "Cheyenne" });
```

- 删除集合及其附带的索引数据
```js
db.users.drop();
```

### 获取帮助信息
```bash
help
```

- 提示如何使用函数
```js
db.help();
```

## 使用索引创建和查询

### 创建大集合
```js
// 插入 20000 条数据
for(let i = 0; i < 20000; i++) {
  db.numbers.save({ num: i });
}

// 查询 numbers 文档的数量
db.numbers.count();

// 查询 numbers 文档
db.numbers.find();

// 输入 it 查看更多
it

// 条件查询
db.numbers.find({ num: 500 });

// 范围查询
// 你可以使用 $gt 和 $lt 运算符来进行范围查询，这两个符号表示大于和小于。
// $gte 表示大于等于 $lte 表示小于等于 $ne 表示不等于
db.numbers.find({ num: { $gt: 20, $lt: 30 } });
```

### 索引和 explain()
```js
// 分析查询语句
db.numbers.find({ num: { $gt: 19995 } }).explain("executionStats");
// totalDocsExamined 字段代表扫描的集合 值为 20000
// totalKeysExamined 字段显示了整个扫描的索引数量 值为 0
// executionTimeMillis 表示查询花费的时间，为 7ms

// 使用 createIndex() 方法为 num 键创建索引
db.numbers.createIndex({ num: 1 });

// 查看文档的索引
db.numbers.getIndexes();

// 分析查询语句
db.numbers.find({ num: { $gt: 19995 } }).explain("executionStats");
// totalDocsExamined 字段代表扫描的集合 值为 4
// totalKeysExamined 字段显示了整个扫描的索引数量 值为 4
// executionTimeMillis 表示查询花费的时间，为 0ms
// 速度上有了极高的提升，从 7ms 变成了 0ms
```

索引并非没有成本，它们会占用空间，而且会让插入成本稍微提升，对于查询优化来说这是必备的工具。

## 基本管理
在 MongoDB 上执行的是非 CRUD 操作，从服务器状态到数据文件完整性检查，都使用数据库命令来实现。

- 打印系统中所有的数据库列表信息：
```js
show dbs
```

- 打印当前数据库里所有的集合
```js
show collections
```

- 分析数据库和集合
```js
db.stats();

db.numbers.stats();
```

### 命令如何执行
```js
// 以下三个命令是等价的
db.numbers.stats();

db.runCommand({ collstats: "numbers" });

db.$cmd.findOne({ collstats: "numbers" });
```

### 获取帮助
```js
// 获取帮助
db.numbers.help();

// 查看实现源码
db.numbers.save
```
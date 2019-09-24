# 面向文档的数据

## 设计电商网站数据模型
- 我们通常推荐为文档创建一个 slug 字段来构建有意义的 URL。这种字段通常唯一索引，以加速查询和确保唯一。
```js
db.products.createIndex({ slug: 1 }, { unique: true });
```

## 核心概念：数据库、集合、文档

### 数据库
- 数据库是集合和索引的命名空间和物理分组。
- MongoDB 没有显式的创建数据库的方式。相反，会在第一次写入数据的时候创建数据库。
```ruby
# 连接数据库
connection = Mongo::Client.new( ['127.0.0.1:27017'], :database => 'garden' )
db = connection.database

# 插入数据
products = db['products'];
products.insert_one({ :name => 'Extra Large Wheelbarrow' });

# 清空集合
products.find({}).delete_many

# 删除集合
products.drop

# 删除数据库
db.drop
```

删除数据库的时候要格外小心，因为没有办法回滚操作，它会从磁盘是删除数据库文件。

- 数据库文件和分配
  - 当创建数据库的时候，MongoDB 会在磁盘是分配一系列数据库文件集合，包括所有的集合、索引，还有其他元数据。
  - mongodb.lock：存储服务器进程 ID；
  - garden.ns：命名空间文件；
  - db.stats()：
    - fileSize：为此数据库分配的总文件空间；
    - dataSize：数据库里实际 BSON 数据的大小；
    - storageSize：额外的为集合增长预留的空间，还有未删除的空间；
    - indexSize：数据库索引的总空间；
- 数据库性能只有在所有使用的索引都加载到内存里才是最好的。

### 集合
集合是机构或概念上相似的文档的容器。

```js
// 显式创建集合
db.createCollection("users");

// 重命名集合
db.products.renameCollection("store_products");

// 盖子集合
db.createCollection("users.actions", { capped: true, size: 16384, max: 100 });

// TTL 集合
db.reviews.createIndex({ time_field: 1, { expireAfterSeconds: 3600 } });

// 查询所有命名空间
db.system.namespaces.find();

// 查询所有索引
db.system.indexes.find();
```

- 盖子集合
  - 盖子集合最初是为高性能日志场景设计的。它与标准的集合不同，因为有固定的大小。这意味着一旦盖子集合到达最大上限，后续的插入将会覆盖最先插入的文档数据。
  - 盖子集合最初是为日志设计的，所以没有实现删除和更新文档操作。
- TTL 集合
  - MongoDB 也允许在特定的时间后废弃文档数据，有时候叫做生存时间 time-to-live（TTL）集合，这个功能实际上是通过一个特殊的索引实现的。
- 系统集合
  - MongoDB 的部分设计依赖于内部集合的使用。
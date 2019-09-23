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
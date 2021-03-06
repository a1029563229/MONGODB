# 构建查询

## 电子商务查询
- 如果想得到单一文件，则当文件存在时 findOne 方法会返回文件；
- 如果我们需要返回多个文件，则使用 find 方法；
- 如果我们使用 findOne 在数据库中查询匹配多个项目，它就会在自然排序文件集合中返回第一个项目；

```js
// findOne
db.products.findOne({ slug: 'wheel-barrow-9092' });

// findOne 方法类似于
db.products.find({ slug: 'wheel-barrow-9092' }).limit(1);
```

### 忽略、限制和排序查询选项
```js
// 分页查询
// 第一页，每页数量 12
db.reviews.find({ 'product_id': product['_id'] }).skip(0).limit(12);

// 排序查询
// 降序排列
db.reviews.find({ 'product_id': product['_id'] }).sort({ 'helpful_votes': -1 }).limit(12);

// 完整示例
// 调用 skip、limit 和 sort 的顺序在 Javascript Shell 中并不重要
page_number = 1;
product = db.products.findOne({ 'slug': 'wheel-barrow-9092' });
category = db.categories.findOne({ '_id': product['main_cat_id'] });
reviews_count = db.reviews.count({ 'product_id': product['_id'] });
reviews = db.reviews.find({ 'product_id': product['_id'] }).sort({ 'helpful_votes': -1 }).skip((page_number - 1) * 12).limit(12);
```
- 这些查找可以使用索引。标签拥有唯一索引，因为它们提供了备用主键，而所有的 _id 字段自动获取标准合计的唯一索引。
- 我们可以依靠索引来处理排序，但是当添加了更多的排序选项时，随着索引数目的增加，维护索引可能变得不太合理，因为每个索引的写入代价都会比较高，这是一个需要权衡利弊的问题。

## 用户与订单
```js
// 使用投影返回限制字段
db.users.findOne({
  'username': 'kbanker',
  'hashed_password': 'bsldknfal1231knl123lbfsbk12'
}, { '_id': 1 })
```

### 用户条件查询
```js
// 正则查询
// 部分正则表达式搜索可以使用索引
db.users.find({ 'last_name': /^Ba/ });

// 查询特定范围
// 为使这种查询有效，需要在指定字段定义索引
db.users.find({ 'addresses.zip': { '$gt': 10019, '$lt': 10040 } });
```

## MongoDB 的查询语言

### 查询条件与选择器
- 范围
  - $lt：小于
  - $gt：大于
  - $lt：小于等于
  - $gte：大于等于
```js
// birthday 大于等于 1985 并且小于等于 2015
db.users.find({ "birthday": { $gte: 1985, $lte: 2015 } });
```

- 设置运算符
  - $in: 如果任意参数在引用集合里，则匹配；频繁使用在 IDs 列表；
  - $nin（not in）：如果没有参数在引用的集合里，则匹配；我们可以使用 $nin 查找所有商品，既不是黑也不是蓝；
  - $all：如果所有参数在引用集合里且被使用在包含数组的文档中，则匹配；如果想找到所有商品标签，如礼品和花园，则 $all 是一个很好的选择；
```js
// 查找分类信息中包含了 a 或 c 的商品
db.mall.find({ "category": { $in: ["a", "c"] } })

// 查找分类信息中不包含 d 的商品
db.mall.find({ "category": { $nin: ["d"] } })

// 查找分类信息中包含 a 和 b 的商品
db.mall.find({ "category": { $in: ["a", "b"] } })
```
这三个查询属性通常用于查询数组项，其中 $in 和 $all 可以使用索引，$nin 最好结合另一个索引属性来进行组合查询。

- 布尔运算符
  - $ne：不匹配参数条件；$ne 适用于关键字指向的单一值或者数组。
  - $not：不匹配结果；$not 不匹配 MongoDB 的运算符或者正则表达式的查询结果。
  - $or：有一个条件匹配就成立；
  - $nor：所有条件都不匹配；
  - $and：所有条件都匹配；
  - $exists：判断元素是否存在；

```js
// $ne 无法使用索引，所以最好结合一个其他索引运算符
// 查找 Acme 制造的所有没有 gardening 标签的商品
db.products.find({ "details.manufacturer": 'Acme', "tags": { $ne: "gardening" } })

// 查找 age 不小于 30 的文档
db.users.find({ "age": { $not: { $lte: 30 } } });

// 如果可能值的范围是相同的关键字，则使用 $in 替代
// 查找是蓝色或者由 Acme 生产的所有商品
// $nor 的用法与 $or 类似
db.users.find({ $or: [
  { 'details.color': 'blue' },
  { 'details.manufacturer': 'Acme' }
] })

// 查找标签是 gift 或 holiday 并且标签为 gardening 或 landscaping
db.products.find({
  $and: [
    {
      tags: { $in: ['gift', 'holiday'] }
    },
    {
      tags: { $in: ['gardening', 'landscaping'] }
    }
  ]
});

// 查询存在颜色字段的文档
db.products.find({ 'details.color': { $exists: true } });
```

### 数组
- 数组查询
  - $elemMatch：如果提供的所有词语在相同的子文档中，则匹配；
  - $size：如果子文档数组大小与提供的文本值相同则，则匹配；
```js
// 查询 name 为 home，state 为 NY 的子文档
db.users.find({
  'addresses': {
    $elemMatch: {
      name: 'home',
      state: 'NY'
    }
  }
})

// 查找拥有三个地址的所有用户
db.users.find({ 'address': { $size: 3 } });
```

### Javascript 查询运算符
- $where 执行任意 Javascript 来选择文档
```js
db.reviews.find({ $where: 'this.helpful_votes > 3' });
```

Javascript 表达式不能使用索引，并且带来大量费用，因为它们必须在 Javascript 解释器的上下文中评估并且是单线程的。所以使用 Javascript 表达式时至少结合一个其他查询运算符，缩小查找的范围。

- $regex：匹配元素对应提供给 regex（正则表达式）项。（无法使用索引）
- $mod[(quotient), (result)]：如果元素除以除数符合结果则匹配；
- $type：如果元素的类型符合指定的 BSON 类型则匹配；
- $text：允许在建立文本索引的字段是执行文本搜索；

## 查询选择

### 映射
- $slice：选择返回文档的子集
```js
// 规定查询选择（返回）的字段为 username
db.users.find({}, { 'username': 1 });

// 排除 addresses 字段和 payment_methods 字段
db.users.find({}, { 'addresses': 0, 'payment_methods': 0 });

// 对子集进行分页处理
db.products.find({}, { 'reviews': { $slice: [0, 10] } });

// 对子集进行分页处理并进行查询选择
db.products.find({}, { 'reviews': { $slice: [0, 10] }, 'reviews.rating': 1 });
```
- sort：排序
- skip + limit：分页（数据过多的分页应使用 $gt + date 字段的组合，因为 skip 在处理大数据时效率比较低下。
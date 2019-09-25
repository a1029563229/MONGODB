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
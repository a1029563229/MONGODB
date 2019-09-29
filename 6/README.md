# 聚合

## 聚合框架概览
- 为调用聚合框架就要定义一个管道。聚合管道里的每一步输出都作为下一步的输入。每一步都在输入文档执行单个操作并生成输出文档。
- 聚合管道操作包含下面几个部分：
  - $project：指定输出文档里的字段（项目化）；
  - $match：选择要处理的文档，与 find() 类似；
  - $limit：限制传递给下一步的文档数量；
  - $skip：跳过一定数量的文档；
  - $unwind：扩展数组，为每个输入入口生成一个输出文档；
  - $group：根据 key 来分组文档；
  - $sort：排序文档；
  - $geoNear：根据某个地理位置附近的文档；
  - $out：把管道的结果写入某个集合；
  - $redact：控制特定数据的访问；

## 电商聚合例子
```js
// 普通方法统计某个商品的评价数量
product = db.products.findOne({ 'slug': 'wheelbarrow-9092' });
reviews_count = db.reviews.count({ 'product_id': product['id'] });

// 聚合框架统计所有商品的评论总数
db.reviews.aggregate([
  {
    $group: {
      _id: '$product_id',
      count: { $sum: 1 }
    }
  }
]);

// 使用聚合框架统计某一商品的评论数量
product = db.products.findOne({ 'slug': 'wheelbarrow-9092' });
db.reviews.aggregate([
  { $match: { product_id: product['_id'] } },
  { $group: { _id: '$product_id', count: { $sum: 1 } } }
]).next();

// 计算商品评价的平均评分
product = db.products.findOne({ 'slug': 'wheelbarrow-9092' });
db.reviews.aggregate([
  { $match: { product_id: product['_id'] } },
  { $group: { 
    _id: '$product_id', 
    average: { $avg: '$rating' },
    count: { $sum: 1 } 
  } }
]).next();

// 细分评分，统计每种评分的数量
product = db.products.findOne({ 'slug': 'wheelbarrow-9092' });
countsByRating = db.reviews.aggregate([
  { $match: { product_id: product['_id'] } },
  { $group: { _id: '$rating', count: { $sum: 1 } } }
]).toArray();

// $unwind 将会为数组里的每个元素生成一个输出文档
// 计算每个类别的商品数量
db.products.aggregate([
  { $project: { category_ids: 1 } },
  { $unwind: '$category_ids' },
  { $group: { _id: '$category_ids', count: { $sum: 1 } } },
  { $out: 'countsByCategory' }
]);
```

### 用户和订单
```js
// 根据用户分组统计每个用户的评价数量，以及有帮助的投票
db.users.aggregate([
  {
    $group: {
      _id: '$user_id',
      count: { $sum: 1 },
      avg_helpful: { $avg: '$helpful_votes' }
    }
  }
]);

// 2010 年根据月和年来统计的订单数据
db.orders.aggregate([
  { $match: { purchase_data: { $gte: new Date(2010, 0, 1) } } },
  { $group: { 
    _id: { year: { $year: '$purchase_data' }, month: { $month: '$purchase_data' } },
    count: { $sum: 1 },
    total: { $sum: '$sub_total' }
   } },
  { $sort: { id: -1 } }
]);
```

- 找出曼哈顿消费最高的用户
```js
// $match 查找快递到曼哈顿的订单
upperManhattanOrders = { 'shipping_address.zip': { $gte: 10019, $lt: 10040 } };

// $group 为每个客户求和
sumByUserId = { id: '$user_id', total: { $sum: '$sub_total' } };

// $match 选择总额超过 100 美元的客户
orderTotalLarge = { total: { $gt: 10000 } };

// $sort 按降序排列结果
sortTotalDesc = { total: -1 };

// 组合查询
db.orders.aggregate([
  { $match: upperManhattanOrders },
  { $group: sumByUserId },
  { $match: orderTotalLarge },
  { $sort: sortTotalDesc }
]);
```

## 聚合管道操作符
- $project
  - 查询映射

```js
// 映射对象
db.users.aggregate([
  { $match: { username: 'kbanker' } },
  { $project: { first_name: 1, last_name: 1 } }
]);



```
- $group
- 主要用于聚合管道。次操作符可以处理多个文档的聚合数据，提供诸多统计功能。
- $addToSet：为组里唯一的值创建一个数组；
- $first：组里的第一个值，只有前缀 $sort 才有意义；
- $last：组里最后一个值，只有前缀 $sort 才有意义；
- $max：组里某个字段的最大值；
- $min：组里某个字段的最小值；
- $avg：某个字段的平均值；
- $push：返回组内所有值的数组。不去除重复值；
- $sum：求组内所有值的和；

```js
// push 的用法
// $push 函数把对象添加到 purchasedItems 数组
db.orders.aggregate([
  { $project: { user_id: 1, line_items: 1 } },
  { $unwind: '$line_items' },
  { $group: { 
    _id: { user_id: '$user_id' },
    purchasedItems: { $push: '$line_items' }
  } }
]).toArray();
```

- $match、$sort、$skip、$limit
```js
db.reviews.aggregate([
  { $match: { 'product_id': product['_id'] } },
  { $skip: (page_number - 1) * 12 },
  { $limit: 12 },
  { $sort: { 'helpful_votes': 1 } }
]).toArray();
```

- $unwind
  - 这个操作符通过为数组里的每一个元素生成一个输出文档来扩展数组。主文档里的字段、每个数组元素的字段都被放入到输出文档里。
```js
db.products.aggregate([
  { $project: { category_ids: 1 } },
  { $unwind: '$category_ids' },
  { $limit: 2 }
]);
```

- $out
  - 保存到指定集合中，通常是最后一个操作符。
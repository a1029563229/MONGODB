# 更新、原子操作和删除

## 文档更新概要

- 通过替换修改

```js
user_id = objectId('xx');
doc = db.users.findOne({ _id: user_id });
doc['email'] = 'mongodb-user@mongodb.com';
db.users.update({ _id: user_id }, doc);
```

- 通过操作符修改
  
```js
user_id = objectId('xx');
db.users.update({ _id: user_id }, {
  $set: { email: 'mongodb-user@mongodb.com' }
});

// 在集合中添加元素
db.products.update({}, { $addToSet: { tags: 'Green' } });

// 更新数量
db.products.update({ _id: product_id }, { $inc: { total_views: 1 } });
```

## 电商数据模型更新

```js
// 更新评论的评分
db.products.update({ _id: product_id }, {
  $set: {
    average_review: average,
    ratings_total: total
  },
  $inc: {
    total_reviews: 1
  }
});

// 增加投票者和投票数量（并且确定该用户没有投过票）
// upsert 将会创建由查询器定位的文档
db.reviews.update({ _id: 'xxx', voter_ids: { $ne: 'xxx' } }, {
  $push: {
    voter_ids: 'xxx'
  },
  $inc: {
    helpful_votes: 1
  }
});

// 统计订单金额
db.orders.update({ user_id: 'xxx' }, { $inc: { sub_total: car_item['pricing'] } }, { upsert: true });

// 在订单中添加商品
db.orders.update(
  { 
    user_id: 'xxx', 
    state: 'CART',
    'line_items._id': { $ne: car_item._id }
  },
  {
    $push: {
      'line_items': cart_item
    }
  }
)
```
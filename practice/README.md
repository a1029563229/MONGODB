# Mongo 查询语句实战练习

## 跨表查询活动用户列表信息

```js
// 不带条件查询
db.getCollection('activity.users')
  .aggregate([
    {
      $lookup: {
        from: "member.users",
        localField: "userId",
        foreignField: "userId",
        as: "u"
      }
    },
    {
      $unwind: {
        path: "$u",
        preserveNullAndEmptyArrays: true
      }
    },
    {
      $addFields: {
        nickname: "$u.nickname",
        avatarUrl: "$u.avatarUrl"
      }
    },
])


// userId 条件
db.getCollection('activity.users')
  .aggregate([
    {
      $lookup: {
        from: "member.users",
        localField: "userId",
        foreignField: "userId",
        as: "u"
      }
    },
    {
      $unwind: {
        path: "$u",
        preserveNullAndEmptyArrays: true
      }
    },
    {
      $addFields: {
        nickname: "$u.nickname",
        avatarUrl: "$u.avatarUrl"
      }
    },
    {
      $match: { nickname: "有效用户" }
    },
    {
      $group: {
        _id:'$userId', 
        count:{$sum:1}
      }
    },
    {
      $skip: 0
    },
    {
      $limit: 10
    },
])
```
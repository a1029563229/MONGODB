# 编写代码操作 MongoDB

## 通过 Ruby lens 连接 MongoDB

- 安装 MongoDB 驱动
```bash
gem install mongo
```

- 连接 MongoDB
```ruby
require 'rubygems'
require 'mongo'

$client = Mongo::Client.new([ '127.0.0.1:27017' ], :database => 'tutorial')
Mongo::Logger.logger.level = ::Logger::ERROR
$users = $client[:users];
puts 'connected!'
```

- 插入数据
```bash
irb -r ./connect.rb

smith = { "last_name" => "smith", "age" => 30 }
jones = { "last_name" => "jones", "age" => 40 }

smith_id = $users.insert_one(smith)
jones_is = $users.insert_one(jones)
```

## 驱动工作原理
- 首先，生成 MongoDB 对象 ID。默认都存储在所有的文档的 _id 字段里。其次，驱动会把任意语言表示的文档对象转换为 BSON 或者从 BSON 转换过来，BSON 是 MongoDB 使用的二进制 JSON 格式。（驱动序列化所有的 Ruby 哈希为 BSON，或者把 BSON 转换为 Ruby 哈希。
- 最后，使用 TCP socket 与数据库连接通信，此时使用的是 MongoDB 自定义协议。socket 通信的风格，特别是写入数据等待应答是非常重要的。
- 每个 MongoDB 文档都需要一个主键。这个键对于每个集合里的文档来说必须是唯一的，存储在文档的 _id 字段里再发送文档数据给服务器之前，驱动会检查是否有 _id 字段。如果没有，就会生产一个对象 _id。
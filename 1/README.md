# 全新 Web 数据库

## MongoDB 健壮性
- MongoDB 的数据模型是面向文档的。
  - MongoDB 以二进制 JSON 格式存储文档数据，或者叫做 BSON；
  - 关系数据库包含表，MongoDB 拥有集合；
  - MySQL 在表的行里保存数据，而 MongoDB 在集合的文档里保存数据；
- 从 MongoDB 2.0 开始，日志功能默认是启用的。启用日志功能后，默认 100 毫秒就会写一次日志文件。如果服务器意外关机，日志会通过重启服务器来确保 MongoDB 数据文件恢复为一致状态。这是运行 MongoDB 最安全的方式。

## 核心服务和工具
- MongoDB 使用 C++ 编写。
- 核心服务器通过名为 mongod 的可执行文件运行。Mongod 服务器进使用自定义的二进制协议（mongodb 协议）在网络上接受命令。所有 mongod 进程的数据库在类 Unix 系统中默认存储在 `/data/db` 路径下。
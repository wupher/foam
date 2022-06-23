# Note of Roomstats

- 超市房态
- 机项盒房态
- 商户通房态

Java 2——3 台
Gearman work 4 台

资源浪费

nats-server

处理方式与商户通相关，比较并非为了事件，而是为了优化 MySQL 处理。其实，使用 KV 库速度会更快。

---
id: on1pkaabrsrx17yvi2tyc9w
title: mongodb_shrink_plan
desc: ''
updated: 1660269713686
created: 1660196760605
---

## 商户通 mongodb 数据自动清理方案

## Idea

之前我们通过运维定时手工清理。费时费力，今年福财事多，直接本来 2 月份的清理计划到 8月份才有时间折腾，而我已经完全忘记这回事了。与此同时，系统仍然照常运行。既然如此，不妨采用类似现在商户通 mysql 数据库推送报表的清理方案，将其做成自动化执行，减少人工参与。

## RTVouchers 自动化清理方案

商户通应用直接新建 4 个 collection：RTVouchers-S1 ~ RTVouchers-S4。代表一年 4 个 season。收到数据时将数据插入到对应的 collection 中。处理数据时也只到对应 seasion 中进行查询即可。

考虑各商家营业时间各不相同，对于同一自然日，最多也就 2 个 seasion 的数据会同时使用。其余 collections 的数据都可以随便清理。运维可编写脚本每季度或每月定时删除，重建非当季 collection 即可。

### 数据迁移

应用上线前，数据存放于统一的 collection `RTVoucher` 中；应用上线后数据存放于各自的 collection 中。因此上线时仅需要将上线那一天的实时账单拷贝到新 collection 即可。可以在应用中提供迁移接口，或通过 mongodb 脚本直接运行均可。

应用也可以提供商家实时数据重捞接口，直接通过 MQ 捞取“当天”营业日数据并存放于对应的 season collection 中。以防止个别商家数据丢失。

## 其它数据清理

除实时账单外，其它一些任务类，日志类数据也需要进行定时清理。可以使用 mysql 数据库清理推送表的方案：每天定时跑脚本进行数据删除。

其它相关需要清楚的数据有：

- datajob: 数据捞取任务
- reCalculationJob: 数据重捞任务
- pushJob: 异步推送任务
- voucherJob: 日报生成任务
- DRGenerateLog: 商户通日报生成错误记录，用于故障跟踪

```js
//delete one years ago's data
var deadLine = new Date(ISODate().getTime() - 1000 * 3600 * 24 * 180);
db.getCollection('dataJob').remove({status:1});
db.getCollection('dataJob').remove({"date" : {$lt: deadLine}});

db.getCollection('reCalculationJob').remove(({status:1}));
db.getCollection('reCalculationJob').remove({"date" : {$lt: deadLine}});

db.getCollection('pushJob').remove({status:1});
```

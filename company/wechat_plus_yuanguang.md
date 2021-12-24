# 微加与大云管 token 数据更新方案查案

## 修订记录

- 2021-12-12: 创建

## 背景

微加服务与大云管之间的数据交换分为两种形式：

- 大云管向微加服务主动发送数据更新请求，如果房态，账单等实时数据更新协议
- 微加服务向大云管发送各种业务请求，如开房，赠送等 MQ 业务协议

数据交换为保证安全，需要在数据交换方维护一个 token 以进行会话签名和加密。本协议即为微加与大云管之间 token 维护 API 设计文档。

## Token 同步协议

与之前小云管不同，微加与大云管不再维护 2 套 Token，而是统一维护一套 token：无论是大云管数据上传，还是微加走 MQ 协议，数据签名及加密都使用同一套 token。微加与大云管之间都提供 token 更新接口，任意一方在发现 token 过期或接近过期均可调用对端接口进行 token 更新。

### Token 更新协议

```yaml
- method: POST
- url: /api/v1/token/update
- request:
  - name: app_id
    type: string
    required: true
    desription: 微加的服务 ID，暂订 10007
  - name: timestamp
    type: integer
    required: true
    desription: unix 时间戳，单位为秒
  - name: sign
    type: string
    required: true
    description: 签名，签名方法为 HMAC-SHA256，签名算法为：
    - 对 app_id 和 timestamp 再加上密钥  进行字符串拼接，拼接后的字符串进行 HMAC-SHA256 签名，得到签名字符串
    - 将签名字符串统一按大写传输
- response:
  - name: token
    type: string
    required: true
    description: 新的 token
  - name: expire_time
    type: string
    required: true
    description: token 过期时间，yyyy-MM-dd HH:mm:ss 格式
```

### 说明

双方的密钥可以不一致，即微加调用大云管使用的 appid/secret 可以和大云管调用微加服务的 appid/secret 不一致。app_id和密钥取得一致后，使用邮件进行沟通，不在协议 API 中提供。

微加对于 token 默认按照 12 小时设置有效期。通过定时程序定时检查 token 是否面临过期，如果 token 过期，或者 token 过期时间检查时间 30 分钟内就直接调用接口向对端申请更新。

在这期间如果收到对端的token 更新请求，如果 token 有效时间大于 30 分钟的阈值，则不做任何处理返回现有 token；如果 token 有效时间小于 30 分钟的阈值，则生成新 token 并更新过期时间。

30 分钟的阈值可通过数据库参数进行调整。

## 大云管商家同步方案

大云管商家的同步方案与小云管机制相同，微加提供增量更新接口，当大云管商家出现新增，关闭等情况调用此接口批量通知微加；大云管同时提供全量接口，当微加需要进行全量数据同步时，调用此接口获取全量商家更新。所有不在全量商家列表内的商家一律处理为关闭。

### 同步接口 大云管——>微加

```yaml
- method: POST
- url: /api/v1/sync/company_update
- request:
  - name: company_list
    type: array
    required: true
    desription: 同步商家列表
    - name: company_code
      type: string
      required: true
      description: 商家编码
    - name: status
      type: integer
      required: true
      description: 商家状态，0 代表商家被关闭大云管功能; 1 代表商家被开启大云管功能
  - name: sign
    type: string
    required: true
    description: 签名，签名方法为 HMAC-SHA256，签名算法为：
    - 根据商家编码和状态拼接成字符串，字符串最后除上当前的大云管 token，拼接后的字符串进行 HMAC-SHA256 签名，得到签名字符串。
    - 将签名字符串统一按大写传输
```

请求正常返回 {ret:0, msg:"OK" }，如果出错，如签名错误，会返回对应参数错误信息。此接口由大云管调用微平台接口。当大云管有商家开通或关闭大云管业务时，通过此接口向微加服务同步商家开通状态。当微加更新状态后，收到的所有 MQ 请求，直接通过 http 接口转发至大云管对应 MQ 接口。

### 同步接口 微加——>大云管 全量同步接口

```yaml
- method: GET
- url: 大云管 API
- headers:
  - name: Authorization
    value: 大云管 token
- response: 返回当前云管有效商家代表
```

微加平台调用此接口用于全量数据同步，当收到返回有商家是大云管商家时，而微家表中的商家又没有对应标识。微加调用此接口与大云管做全量商家同步，根据返回的数据重新更新所有大云管商家。

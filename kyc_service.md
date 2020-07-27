# KYC Service

## 概述

KYC Service 是火币公链的KYC需求的实现.

KYC机构的正常使用场景
1. KYC机构经过线下渠道向火币申请开启KYC服务,并获得火币的批批准
2. 用户通过线下渠道,向某个KYC机构提交KYC的资质证明
3. KYC机构审核用户的资料后,在火币公链的KYC服务里,更新该用户的信息,这里主要的操作是更新该用户的TAG
4. 金融合作伙伴编写金融产品的智能合约,并在合约编写业务逻辑,包括针对用户所需求的KYC机构和该机构下的TAG
5. 金融合作伙伴线下向火币申请部署金融产品上链,火币审查通过后,该金融产品上链上线.(该步骤不属于KYC需求)
6. 用户在该金融产品下进行金融活动,金融产品触发业务逻辑,向KYC服务发送请求,并完成后续业务

用户向KYC服务提交新的资质证明,更新资质
1. 用户线下向KYC机构提交新的资质证明
2. KYC机构审核后,向KYC服务发送 Transaction,更新用户的TAG

KYC机构主动修改用户资质
1. KYC机构自行向KYC服务服务发送 Transaction,更新用户的TAG

金融合作伙伴更改KYC需求
1. 金融合作伙伴编写新的金融产品智能合约
2. 金融合作伙伴在火币的监管下,在火币公链个更新智能合约

## KYC表达式

由于调用KYC需求的金融产品智能合约有随时改变KYC条件的业务诉求,但是确不需要改动智能合约的业务逻辑.所以有了基于KYC表达式的解决方案.

金融产品智能合约可以通过一个可以随时修改的KYC表达式,来向KYC服务请求表达式的计算.最后根据KYC表达式的结果,来决定下一步业务步骤.

## KYC表达式规范

### 标签格式

- 标签是由KEY:[VALUE1,VALUE2,....VALUE16]的形式存在的.

- KEY是ASCII字符串,字母开头,后续可以是字母,数字,下划线.总长度不超过32个ASCII字符.

- VALUE是ASCII字符串,字母开头,后续可以是字母,数字,下划线.总长度不超过32个ASCII字符.用'`'符号标记,也就是acute符号.

- [VALUE1,VALUE2,....VALUE16]的总长度不超过16,但是必须有一个值.如果值不存在,应当删除这个KEY

- 每个KYC服务都有各自的标签空间,不同的KYC服务的标签互相不影响.

- 每一个KYC服务下,KEY不能重复.

- 每一个KEY下,VALUE不能重复.

- 如果一个KEY不存在,那么对应的VALUE默认为NULL,NULL是VALUE的保留关键字,在设定标签的时候不能给出NULL这个值.

### 标签分类

标签在概念上分为"信息类"标签和"评价类"标签两大类.

- 信息类标签通常是用户向KYC机构提供自身身份信息类的数据,KYC机构认为有必要在KYC服务上给予特定的标签.这类标签原则上不可以被更新.
  - 这一版暂时不约束标签修改.

- 评价类标签通常是KYC机构根据自身业务对用户的评价,这类标签可以也只能由该KYC机构更新.

### expression表达式的结构
- org_name.TAG 定义需求的kyc服务和需求的TAG,例如HUOBI-KYC.NATION
- 运算符 @ 表示"一个TAG内需要一个VALUE",例如HUOBI-KYC.NATION@"US"
- 运算符 () 用以调整优先级
- 运算符 && 和 || 用以逻辑计算
- 运算符 ! 用以反转逻辑结果
- 表达式只能计算一个用户地址下的KYC标签,不能同时计算多个用户的表达式.
- 示例 !HUOBI-KYC.NATION@`CN` && HUOBI-KYC.GENDER@`MALE`,国籍不能是中国,性别是男性
- 示例 HUOBI-KYC.ADDRESS@`NULL`,地址的TAG不存在

## 接口

### 1. 获得所有已注册的KYC机构

```rust
#[read]
    fn get_orgs(&self, ctx: ServiceContext, _: String) -> ServiceResponse<Vec<OrgName>>{}
```

GraphiQL 示例：

```graphql
query status{
  queryService(
  caller: "0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "kyc"
  method: "get_orgs"
  payload:"",
  ){
    ret,
    isError
  }
}
```

### 2. 获得一个KYC机构的信息.

```rust
#[read]
    fn get_org_info(&self, ctx: ServiceContext, org_name: OrgName, ) -> ServiceResponse<Option<KycOrgInfo>> {}
```

GraphiQL 示例：

```graphql
query status{
  queryService(
  caller: "0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "kyc"
  method: "get_org_info"
  payload:"your org name",
  ){
    ret,
    isError
  }
}
```

### 3. 获得一个KYC机构声明他所支持的TAG.

```rust
#[read]
    fn get_org_supported_tags( &self, ctx: ServiceContext, org_name: OrgName,) -> ServiceResponse<Vec<TagName>>{}
```

GraphiQL 示例：

```graphql
query status{
  queryService(
  caller: "0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "kyc"
  method: "get_org_supported_tags"
  payload:"your org name",
  ){
    ret,
    isError
  }
}
```

### 4. 获得一个用户 Address 在某个KYC机构下的TAG信息.

```rust
#[read]
    fn get_user_tags( &self, ctx: ServiceContext, payload: GetUserTags, ) -> ServiceResponse<HashMap<TagName, FixedTagList>>{}
```

GraphiQL 示例：

```graphql
query status{
  queryService(
  caller: "0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "kyc"
  method: "get_user_tags"
  payload:"{\"org_name\":\"your org name\",\"user\":\"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\"}",
  ){
    ret,
    isError
  }
}
```

### 5. 基于一个用户 Address,运行一个KYC表达式.

```rust
#[read]
    fn eval_user_tag_expression( &self, ctx: ServiceContext, payload: EvalUserTagExpression, ) -> ServiceResponse<bool>{}
```

GraphiQL 示例：

```graphql
query status{
  queryService(
  caller: "0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "kyc"
  method: "eval_user_tag_expression"
  payload:"{\"user\":\"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\",\"expression\":\"KYC expression, more details later\"}",
  ){
    ret,
    isError
  }
}
```

### 6. 批准/取消批准一个KYC机构

```rust
#[write]
    fn change_org_approved( &mut self, ctx: ServiceContext, payload: ChangeOrgApproved, ) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"kyc",
    method:"change_org_approved",
    payload:"{\"org_name\": \"your org name\",\"approved\": false}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

### 7. 修改KYC服务的 admin

```rust
#[write]
    fn change_service_admin( &mut self, ctx: ServiceContext, new_admin: Address, ) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"kyc",
    method:"change_service_admin",
    payload:"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

### 8. 修改KYC机构的 admin

```rust
#[write]
    fn change_org_admin( &mut self, ctx: ServiceContext, payload: ChangeOrgAdmin, ) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"kyc",
    method:"change_org_admin",
    payload:"{\"name\": \"your org name\",\"new_admin\": \"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\"}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

### 9. 注册一个KYC机构

```rust
#[write]
    fn register_org( &mut self, ctx: ServiceContext, new_org: RegisterNewOrg, ) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"kyc",
    method:"register_org",
    payload:"{\"name\": \"your org name\",\"description\": \"bala bala\",\"admin\": \"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\",\"supported_tags\":[\"Tag1\",\"Tag2\"]}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

### 10. KYC机构更新自身支持的TAG

```rust
#[write]
    fn update_supported_tags( &mut self, ctx: ServiceContext, payload: UpdateOrgSupportTags, ) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"kyc",
    method:"update_supported_tags",
    payload:"{\"org_name\": \"your org name\",\"supported_tags\":[\"Tag1\",\"Tag2\"]}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

### 11. KYC机构更新某一个用户 Address 的TAG

```rust
#[write]
    fn update_user_tags( &mut self, ctx: ServiceContext, payload: UpdateUserTags, ) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"kyc",
    method:"update_user_tags",
    payload:"{\"org_name\": \"your org name\",\"user\":\"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\",\"tags\":{\"Tag1\":[\"Val1\",\"Val2\"]}}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```
# Admission Control Service

## 概述

Admission Control Service 是火币公链的准入控制服务.

所谓准入系统,指的是控制transaction能否进入 mempool 的系统.如果一笔 tx 无法进入 mempool,那也就意味着它无法被共识,更无法被执行.

Admission Control Service 的部分方法会被注册到 Authorization Service,在 tx 进入 mempool 的时候,这些被注册的方法,会被一次调用.
就像有一个漏斗一样,tx会被一层一层地过滤筛选.最后通过筛选的方法才能进入 mempool.

Admission Control Service 主要提供2种被校验的方法.

1. is_valid(),校验当前tx的caller的余额是否充足,是否能满足 tx_failure_fee.
2. is_permitted(),校验当前的 tx 的 caller 是否在禁止名单内.

## 接口

### 1. 检查一笔tx是否能通过禁止名单.

这不是一个普通接口,这个方法应该被 mempool 校验交易所调用.但是你也可以单独调用他用以测试.

```rust
#[read]
fn is_permitted(&self, ctx: ServiceContext, payload: SignedTransaction) -> ServiceResponse<()>{}
```

GraphiQL 示例：

N/A,建议通过 GraphQL 调用.


### 2. 检查一笔tx的caller的余额,是否能满足交易失败手续费.

这不是一个普通接口,这个方法应该被mempool校验交易所调用.但是你也可以单独调用他用以测试.

```rust
#[read]
    fn is_valid(&self, ctx: ServiceContext, payload: SignedTransaction) -> ServiceResponse<()>{}
```

GraphiQL 示例：
N/A,建议通过 GraphQL 调用.

### 3. 获取一系列 address ,是否在禁止名单内.返回的是在禁止名单的地址.

```rust
#[read]
    fn status(&self, _ctx: ServiceContext, payload: AddressList) -> ServiceResponse<StatusList>{}
```

GraphiQL 示例：

```graphql
query status{
  queryService(
  caller: "0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "admission_control"
  method: "status"
  payload:"{\"addrs\": [\"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\"]}",
  ){
    ret,
    isError
  }
}
```

### 4. 修改Admission Control Service的Admin

```rust
#[write]
    fn change_admin(&mut self, ctx: ServiceContext, payload: NewAdmin) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"admission_control",
    method:"change_admin",
    payload:"{\"new_admin\": \"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\"}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

### 5. 将一个地址列入forbid list

```rust
#[write]
    fn forbid(&mut self, ctx: ServiceContext, payload: AddressList) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"admission_control",
    method:"forbid",
    payload:"{\"addrs\": \"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\"}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

### 6. 将一个地址从 deny list 内移除

```rust
#[write]
    fn permit(&mut self, ctx: ServiceContext, payload: AddressList) -> ServiceResponse<()>{}
```

GraphiQL 示例：

```graphql
mutation update_metadata{
  unsafeSendTransaction(inputRaw: {
    serviceName:"admission_control",
    method:"permit",
    payload:"{\"addrs\": \"0x016cbd9ee47a255a6f68882918dcdd9e14e6bee1\"}",
    timeout:"0xbe",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```
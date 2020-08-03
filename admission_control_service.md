# Admission Control Service

## 概述

Admission Control Service 是火币公链的准入控制服务。

所谓准入系统，指的是控制transaction能否进入 mempool 的系统。如果一笔 tx 无法进入 mempool，那也就意味着它无法被共识，更无法被执行。

Admission Control Service 的部分方法会被注册到 Authorization Service，在 tx 进入 mempool 的时候，这些被注册的方法，会被一次调用。
就像有一个漏斗一样，tx会被一层一层地过滤筛选，最后通过筛选的方法才能进入 mempool。

Admission Control Service 主要提供2种被校验的方法。

1. is_valid()，校验当前tx的caller的余额是否充足，是否能满足 tx_failure_fee。
2. is_permitted()，校验当前的 tx 的 caller 是否在禁止名单内。

## 接口

### 1. 检查一笔 tx 是否能通过禁止名单。

这不是一个普通接口，这个方法应该被 mempool 校验交易所调用。但是你也可以单独调用他用以测试。

```rust
#[read]
fn is_permitted(&self, ctx: ServiceContext, payload: SignedTransaction) -> ServiceResponse<()>{}
```

### 2. 检查一笔 tx 的 caller 的余额，是否能满足交易失败手续费。

这不是一个普通接口，这个方法应该被mempool校验交易所调用。但是你也可以单独调用他用以测试。

```rust
#[read]
    fn is_valid(&self, ctx: ServiceContext, payload: SignedTransaction) -> ServiceResponse<()>{}
```

### 3. 获取一系列 address，是否在禁止名单内。返回的是在禁止名单的地址。

```rust
#[read]
    fn status(&self, _ctx: ServiceContext, payload: AddressList) -> ServiceResponse<StatusList>{}
```

### 4. 修改 Admission Control Service 的 Admin

```rust
#[write]
    fn change_admin(&mut self, ctx: ServiceContext, payload: NewAdmin) -> ServiceResponse<()>{}
```

### 5. 将一个地址列入 deny list，如果该地址已经在 deny list 中，则不做任何操作

```rust
#[write]
    fn forbid(&mut self, ctx: ServiceContext, payload: AddressList) -> ServiceResponse<()>{}
```

### 6. 将一个地址从 deny list 内移除，如果该地址不在 deny list 中，则不做任何操作

```rust
#[write]
    fn permit(&mut self, ctx: ServiceContext, payload: AddressList) -> ServiceResponse<()>{}
```

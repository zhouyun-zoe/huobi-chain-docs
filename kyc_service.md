# KYC Service

## 概述

KYC Service 旨在为火币公链提供 KYC 服务.

KYC 机构的正常使用场景是：

1. KYC 机构经过线下渠道向火币申请开启 KYC 服务，并获得火币的批批准
2. 用户通过线下渠道，向某个 KYC 机构提交 KYC 的资质证明
3. KYC 机构审核用户的资料后，在火币公链的 KYC 服务里，更新该用户的信息，这里主要的操作是更新该用户的 TAG
4. 金融合作伙伴编写金融产品的智能合约，并在合约编写业务逻辑，包括针对用户所需求的 KYC 机构和该机构下的 TAG
5. 金融合作伙伴线下向火币申请部署金融产品上链，火币审查通过后，该金融产品上链上线。(该步骤不属于 KYC 需求)
6. 用户在该金融产品下进行金融活动，金融产品触发业务逻辑，向 KYC 服务发送请求，并完成后续业务

用户向 KYC 服务提交新的资质证明，更新资质：
1. 用户线下向 KYC 机构提交新的资质证明
2. KYC 机构审核后，向 KYC 服务发送 Transaction，更新用户的 TAG

KYC 机构主动修改用户资质：
1. KYC 机构自行向 KYC 服务服务发送 Transaction，更新用户的 TAG

金融合作伙伴更改 KYC：
1. 金融合作伙伴编写新的金融产品智能合约
2. 金融合作伙伴在火币的监管下，在火币公链个更新智能合约

## KYC 表达式

由于调用 KYC 需求的金融产品智能合约有随时改变 KYC 条件的业务诉求，但是确不需要改动智能合约的业务逻辑。所以有了基于 KYC 表达式的解决方案。

金融产品智能合约可以通过一个可以随时修改的 KYC 表达式，来向 KYC 服务请求表达式的计算。最后根据 KYC 表达式的结果，来决定下一步业务步骤。

## KYC 表达式规范

### 标签格式

- 标签是由 KEY:[VALUE1，VALUE2，....VALUE16] 的形式存在的。

- KEY 是 ASCII 字符串，字母开头，后续可以是字母，数字，下划线。总长度不超过 32 个 ASCII 字符。

- VALUE 是 ASCII 字符串，字母开头，后续可以是字母，数字，下划线。总长度不超过 32 个 ASCII 字符。用 '`' 符号标记，也就是 acute 符号。

- [VALUE1，VALUE2，....VALUE16] 的总长度不超过 16，但是必须有一个值。如果值不存在，应当删除这个 KEY。

- 每个 KYC 服务都有各自的标签空间，不同的 KYC 服务的标签互相不影响。

- 每一个 KYC 服务下，KEY 不能重复。

- 每一个 KEY 下，VALUE 不能重复。

- 如果一个 KEY 不存在，那么对应的 VALUE 默认为 NULL，NULL 是 VALUE 的保留关键字，在设定标签的时候不能给出 NULL 这个值。

### 标签分类

标签在概念上分为”信息类“标签和”评价类“标签两大类。

- 信息类标签通常是用户向 KYC 机构提供自身身份信息类的数据，KYC 机构认为有必要在 KYC 服务上给予特定的标签。这类标签原则上不可以被更新。
  - 这一版暂时不约束标签修改.

- 评价类标签通常是 KYC 机构根据自身业务对用户的评价，这类标签可以也只能由该 KYC 机构更新。

### expression 表达式的结构

- org_name.TAG 定义需求的 kyc 服务和需求的 TAG，例如 HUOBI-KYC.NATION
- 运算符 @ 表示”一个 TAG 内需要一个VALUE“，例如 HUOBI-KYC.NATION@"US"
- 运算符 () 用以调整优先级
- 运算符 && 和 || 用以逻辑计算
- 运算符 ! 用以反转逻辑结果
- 表达式只能计算一个用户地址下的 KYC 标签，不能同时计算多个用户的表达式
- 示例 !HUOBI-KYC.NATION@`CN` && HUOBI-KYC.GENDER@`MALE`，国籍不能是中国，性别是男性
- 示例 HUOBI-KYC.ADDRESS@`NULL`，地址的 TAG 不存在

## 接口

### 1. 获得所有已注册的 KYC 机构

```rust
#[read]
fn get_orgs(&self, ctx: ServiceContext) -> ServiceResponse<Vec<OrgName>>{}
```

### 2. 获得一个 KYC 机构的信息

```rust
#[read]
fn get_org_info(&self, ctx: ServiceContext, org_name: OrgName, ) -> ServiceResponse<Option<KycOrgInfo>> {}

pub struct KycOrgInfo {
    pub name:           OrgName,
    pub description:    String,
    pub admin:          Address,
    pub supported_tags: Vec<TagName>,
    pub approved:       bool,
}
```

### 3. 获得一个 KYC 机构声明他所支持的 TAG

```rust
#[read]
fn get_org_supported_tags( &self, ctx: ServiceContext, org_name: OrgName,) -> ServiceResponse<Vec<TagName>>{}
```

### 4. 获得一个用户 Address 在某个 KYC 机构下的 TAG 信息

```rust
#[read]
fn get_user_tags( &self, ctx: ServiceContext, payload: GetUserTags, ) -> ServiceResponse<HashMap<TagName, FixedTagList>>{}

pub struct GetUserTags {
    pub org_name: OrgName,
    pub user:     Address,
}

```

### 5. 基于一个用户 Address，运行一个 KYC 表达式

```rust
#[read]
fn eval_user_tag_expression( &self, ctx: ServiceContext, payload: EvalUserTagExpression, ) -> ServiceResponse<bool>{}

pub struct EvalUserTagExpression {
    pub user:       Address,
    pub expression: String,
}
```


### 6. 批准/取消批准一个 KYC 机构

```rust
#[write]
fn change_org_approved( &mut self, ctx: ServiceContext, payload: ChangeOrgApproved, ) -> ServiceResponse<()>{}

pub struct ChangeOrgApproved {
    pub org_name: OrgName,
    pub approved: bool,
}
```

### 7. 修改 KYC 服务的 admin

```rust
#[write]
fn change_service_admin(
        &mut self,
        ctx: ServiceContext,
        payload: ChangeServiceAdmin,
    ) -> ServiceResponse<()>{}

pub struct ChangeServiceAdmin {
    pub new_admin: Address,
}
```

### 8. 修改 KYC 机构的 admin

```rust
#[write]
fn change_org_admin( &mut self, ctx: ServiceContext, payload: ChangeOrgAdmin, ) -> ServiceResponse<()>{}

pub struct ChangeOrgAdmin {
    pub name:      OrgName,
    pub new_admin: Address,
}
```

### 9. 注册一个 KYC 机构

```rust
#[write]
fn register_org( &mut self, ctx: ServiceContext, new_org: RegisterNewOrg, ) -> ServiceResponse<()>{}

pub struct RegisterNewOrg {
    pub name:           OrgName,
    pub description:    String,
    pub admin:          Address,
    pub supported_tags: Vec<TagName>,
}
```

### 10. KYC 机构更新自身支持的TAG

```rust
#[write]
fn update_supported_tags( &mut self, ctx: ServiceContext, payload: UpdateOrgSupportTags, ) -> ServiceResponse<()>{}

pub struct UpdateOrgSupportTags {
    pub org_name:       OrgName,
    pub supported_tags: Vec<TagName>,
}
```

### 11. KYC 机构更新某一个用户 Address 的 TAG

```rust
#[write]
fn update_user_tags( &mut self, ctx: ServiceContext, payload: UpdateUserTags, ) -> ServiceResponse<()>{}

pub struct UpdateUserTags {
    pub org_name: OrgName,
    pub user:     Address,
    pub tags:     HashMap<TagName, FixedTagList>,
}
```

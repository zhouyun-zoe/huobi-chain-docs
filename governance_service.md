# Governance Service

## 概述

Governance Service 负责节点的共识配置以及权限的管理（metadata）和手续费计费以及矿工费的支付。

Governance Service 对共识配置的管理通过更改 [metadata](./metadata_service.md) 进行。metadata 中包括共识列表，出块时间，共识每阶段的超时配置，`cycle_limit`，`cycle_price`，每个块中的最大交易数等等。

在每一笔交易链路中，每当执行过程中产生利润，都要在 Governance Service 中进行利润申报，当每一笔交易执行完成后，Governance Service 会根据交易发起人账户余额和所有利润之和计算手续费金额并进行扣费。当每一个区块包含的所有的交易执行完成后，会向出块人的账户支付矿工费。

## 接口

1.读取 admin 地址

```rust
fn get_admin(&self, ctx: ServiceContext) -> ProtocolResult<Address>;
```

2.设置 admin 地址

```rust
// 需要 admin 权限
fn set_admin(&mut self, ctx: ServiceContext, payload: SetAdminPayload) -> ProtocolResult<()>;

// 参数
pub struct SetAdminPayload {
    pub admin: Address,
}
```

3.更新元数据
   
```rust
// 需要 admin 权限
fn update_metadata(&mut self, ctx: ServiceContext, payload: UpdateMetadataPayload) -> ProtocolResult<()>;

// 参数
pub struct UpdateMetadataPayload {
    pub verifier_list:   Vec<ValidatorExtend>,
    pub interval:        u64,
    pub propose_ratio:   u64,
    pub prevote_ratio:   u64,
    pub precommit_ratio: u64,
    pub brake_ratio:     u64,
    pub timeout_gap:     u64,
    pub cycles_limit:    u64,
    pub cycles_price:    u64,
    pub tx_num_limit:    u64,
    pub max_tx_size:     u64,
}

pub struct ValidatorExtend {
    pub bls_pub_key:    Hex,
    pub pub_key:        Hex,
    pub address:        Address,
    pub propose_weight: u32,
    pub vote_weight:    u32,
}
```

4.更新区块间隔

```rust
// 需要 admin 权限
fn update_interval(&mut self, ctx: ServiceContext, payload: UpdateIntervalPayload) -> ProtocolResult<()>;

// 参数
pub struct UpdateIntervalPayload {
    pub interval: u64,
}
```

5.更新验证人集合

```rust
// 需要 admin 权限
fn update_validators(&mut self, ctx: ServiceContext, payload: UpdateValidatorsPayload) -> ProtocolResult<()>;

// 参数
pub struct UpdateValidatorsPayload {
    pub verifier_list: Vec<ValidatorExtend>,
}

pub struct ValidatorExtend {
    pub bls_pub_key:    Hex,
    pub pub_key:        Hex,
    pub address:        Address,
    pub propose_weight: u32,
    pub vote_weight:    u32,
}
```

6.更新共识 round 超时时间

```rust
// 需要 admin 权限
fn update_ratio(&mut self, ctx: ServiceContext, payload: UpdateRatioPayload) -> ProtocolResult<()>;

// 参数
pub struct UpdateRatioPayload {
    pub propose_ratio:   u64,
    pub prevote_ratio:   u64,
    pub precommit_ratio: u64,
    pub brake_ratio: u64
}
```

7.获取治理信息

```rust
fn get_govern_info(&self, ctx: ServiceContext) -> ServiceResponse<GovernanceInfo>;

// 参数
pub struct GovernanceInfo {
    pub admin: Address, 
    // 交易失败手续费。每一笔交易在运行开始，都需要先扣除交易失败手续费。
    // 每一笔交易在进入 mempool 时，都需要满足 sender 的余额大于交易失败手续费
    pub tx_failure_fee: u64,
    pub tx_floor_fee: u64, // 交易成功时，收取的最低手续费
    pub profit_deduct_rate_per_million: u64, // 交易产生利润时的费率，注意是百万分之X
    pub tx_fee_discount: Vec<DiscountLevel>, // 手续费折扣率
    pub miner_benefit: u64, // 矿工的出块奖励
}
```

8.设置治理信息

```rust
// 需要 admin 权限
fn set_govern_info(&mut self, ctx: ServiceContext, payload: SetGovernInfoPayload) -> ServiceResponse<()>;

// 参数
pub struct SetGovernInfoPayload {
    pub inner: GovernanceInfo,
}

pub struct GovernanceInfo {
    pub admin: Address,
    pub tx_failure_fee: u64,
    pub tx_floor_fee: u64,
    pub profit_deduct_rate_per_million: u64,
    pub tx_fee_discount: Vec<DiscountLevel>,
    pub miner_benefit: u64,
}
```

9.获取交易失败手续费

```rust
fn get_tx_failure_fee(&self, ctx: ServiceContext) -> ServiceResponse<u64>;
```

10.获取最低手续费

```rust
fn get_tx_floor_fee(&self, ctx: ServiceContext) -> ServiceResponse<u64>;
```

11.设置矿工收款地址

```rust
// 需要 admin 权限
fn set_miner(&mut self, ctx: ServiceContext, payload: MinerChargeConfig) -> ServiceResponse<()>;

// 参数
pub struct MinerChargeConfig {
    // 矿工地址
    pub address:              Address,
    // 矿工收款地址
    pub miner_charge_address: Address,
}
```

12.申报利润

```rust
fn accumulate_profit(&mut self, ctx: ServiceContext, payload: AccmulateProfitPayload) -> ServiceResponse<()>;

// 参数
pub struct AccmulateProfitPayload {
    // 获得利润的地址，目前仅作为记录
    pub address:            Address,
    // 获得的利润
    pub accumulated_profit: u64,
}
```

13.注意
交易手续费的扣除会在每一笔交易执行结束之后自动进行，矿工费的支付会在每一个区块的交易全部执行结束之后进行。具体的交易手续费的计算规则参见[手续费计算](./calc_fee.md)。

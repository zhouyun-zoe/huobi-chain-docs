# Huobi-chain 入门

<details>
  <summary><strong>Table of Contents</strong></summary>

- [Huobi-chain 入门](#huobi-chain-%e5%85%a5%e9%97%a8)
  - [安装和运行](#%e5%ae%89%e8%a3%85%e5%92%8c%e8%bf%90%e8%a1%8c)
    - [安装依赖](#%e5%ae%89%e8%a3%85%e4%be%9d%e8%b5%96)
      - [** MacOS **](#macos)
      - [** ubuntu **](#ubuntu)
      - [** centos7 **](#centos7)
      - [** archlinux **](#archlinux)
    - [直接下载预编译的二进制文件](#%e7%9b%b4%e6%8e%a5%e4%b8%8b%e8%bd%bd%e9%a2%84%e7%bc%96%e8%af%91%e7%9a%84%e4%ba%8c%e8%bf%9b%e5%88%b6%e6%96%87%e4%bb%b6)
    - [从源码编译](#%e4%bb%8e%e6%ba%90%e7%a0%81%e7%bc%96%e8%af%91)
      - [获取源码](#%e8%8e%b7%e5%8f%96%e6%ba%90%e7%a0%81)
      - [安装 RUST](#%e5%ae%89%e8%a3%85-rust)
      - [编译](#%e7%bc%96%e8%af%91)
    - [运行单节点](#%e8%bf%90%e8%a1%8c%e5%8d%95%e8%8a%82%e7%82%b9)
  - [与链进行交互](#%e4%b8%8e%e9%93%be%e8%bf%9b%e8%a1%8c%e4%ba%a4%e4%ba%92)
    - [使用 GraphiQL 与链进行交互](#%e4%bd%bf%e7%94%a8-graphiql-%e4%b8%8e%e9%93%be%e8%bf%9b%e8%a1%8c%e4%ba%a4%e4%ba%92)
    - [使用 muta-cli 与链进行交互](#%e4%bd%bf%e7%94%a8-muta-cli-%e4%b8%8e%e9%93%be%e8%bf%9b%e8%a1%8c%e4%ba%a4%e4%ba%92)
  - [使用示例](#%e4%bd%bf%e7%94%a8%e7%a4%ba%e4%be%8b)
  - [使用 docker 本地部署多节点链](#%e4%bd%bf%e7%94%a8-docker-%e6%9c%ac%e5%9c%b0%e9%83%a8%e7%bd%b2%e5%a4%9a%e8%8a%82%e7%82%b9%e9%93%be)
  - [通过 sdk 手动构造交易的方法](#%e9%80%9a%e8%bf%87-sdk-%e6%89%8b%e5%8a%a8%e6%9e%84%e9%80%a0%e4%ba%a4%e6%98%93%e7%9a%84%e6%96%b9%e6%b3%95)

  </details>

## 安装和运行

### 安装依赖

<!-- tabs:start -->

#### ** MacOS **

```
$ brew install autoconf libtool
```

#### ** ubuntu **

```
$ apt update
$ apt install -y git curl openssl cmake pkg-config libssl-dev gcc build-essential clang libclang-dev
```

#### ** centos7 **

```
$ yum install -y centos-release-scl
$ yum install -y git make gcc-c++ openssl-devel llvm-toolset-7

# 打开 llvm 支持
$ scl enable llvm-toolset-7 bash
```

#### ** archlinux **

```
$ pacman -Sy --noconfirm git gcc pkgconf clang make
```

<!-- tabs:end -->

### 从源码编译

#### 获取源码

通过 git 下载源码：

```
$ git clone https://github.com/HuobiGroup/huobi-chain.git
```

或者在 [github releases](https://github.com/HuobiGroup/huobi-chain/releases) 下载源码压缩包解压。

#### 安装 RUST

参考： <https://www.rust-lang.org/tools/install>

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### 编译

```
$ cd /path/to/huobi-chain
$ make prod
```

编译完成后的二进制文件在 `target/release/huobi-chain`。

### 运行单节点

```bash
$ cd /path/to/huobi-chain

# 使用默认配置运行 huobi-chain
# 如果是直接下载的 binary，请自行替换下面的命令为对应的路径
./target/release/huobi-chain

# 查看帮助
$ ./target/release/huobi-chain  -h
Huobi-chain v0.3.0
Muta Dev <muta@nervos.org>

USAGE:
    huobi-chain [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -c, --config <FILE>     a required file for the configuration [default: ./config/chain.toml]
    -g, --genesis <FILE>    a required file for the genesis [default: ./config/genesis.toml]
```

## 与链进行交互

链默认在 8000 端口暴露了 GraphQL 接口用于用户与链进行交互。

### 使用 GraphiQL 与链进行交互

打开 <http://127.0.0.1:8000/graphiql> 后效果如下图所示：

![](./static/graphiql.png)

左边输入 GraphQL 语句，点击中间的执行键，即可在右边看到执行结果。

点击右边 Docs 可以查阅接口文档。更多 GraphQL 用法可以参阅 [官方文档](https://graphql.org/)。

### 使用 muta-cli 与链进行交互

我们通过 [muta-sdk](./js_sdk) 和 nodejs 封装了一个交互式命令行，可以更方便的与 huobi-chain 进行交互。

```bash
$ npm install -g muta-cli@0.2.0-dev.2

$ muta-cli repl
> await client.getLatestBlockHeight()
2081
> await client.getBlock('0x1')
{
  header: {
    chainId: '0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036',
    confirmRoot: [],
    cyclesUsed: [],
    execHeight: '0x0000000000000000',
    height: '0x0000000000000001',
    orderRoot: '0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421',
    orderSignedTransactionsHash: '0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421',
    prevHash: '0x1f83ad5a7ab75787057fddff4d8783340e52315e1f1919a6e06072f2f568fa81',
    proof: {
      bitmap: '0x',
      blockHash: '0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421',
      height: '0x0000000000000000',
      round: '0x0000000000000000',
      signature: '0x'
    },
    proposer: '0xf8389d774afdad8755ef8e629e5a154fddc6325a',
    receiptRoot: [],
    stateRoot: '0x14c8cbb4bdd9d2047313f639e2139057d19b5dad81b45806199c1b4147b9bb38',
    timestamp: '0x000001736a57027e',
    validatorVersion: '0x0000000000000000',
    validators: [ [Object] ]
  },
  orderedTxHashes: [],
  hash: '0xc477e292e6ce9fdaa2a4487261d42296dd47b1d3c789bb3bf242ab2ac038e5ac'
}
```

该 REPL 是基于 nodejs 的封装，你可以使用任何符合 nodejs 语法的语句。更多使用方法可以参考 [muta-sdk 文档](https://nervosnetwork.github.io/muta-sdk-js/)

## 使用示例

以下使用 muta-cli 对链的常用操作进行简单的示例说明：

```bash
$ muta-cli repl
# 链基础交互
> await client.getLatestBlockHeight()
2081

> client.getBlock('0x1')
{
  header: {
    chainId: '0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036',
    confirmRoot: [],
    cyclesUsed: [],
    execHeight: '0x0000000000000000',
    height: '0x0000000000000001',
    orderRoot: '0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421',
    orderSignedTransactionsHash: '0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421',
    prevHash: '0x1f83ad5a7ab75787057fddff4d8783340e52315e1f1919a6e06072f2f568fa81',
    proof: {
      bitmap: '0x',
      blockHash: '0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421',
      height: '0x0000000000000000',
      round: '0x0000000000000000',
      signature: '0x'
    },
    proposer: '0xf8389d774afdad8755ef8e629e5a154fddc6325a',
    receiptRoot: [],
    stateRoot: '0x14c8cbb4bdd9d2047313f639e2139057d19b5dad81b45806199c1b4147b9bb38',
    timestamp: '0x000001736a57027e',
    validatorVersion: '0x0000000000000000',
    validators: [ [Object] ]
  },
  orderedTxHashes: [],
  hash: '0xc477e292e6ce9fdaa2a4487261d42296dd47b1d3c789bb3bf242ab2ac038e5ac'
}

# asset service 操作

# 起链后只有 admin 的账户有 HTTest，由于后续的一些写操作需要以 HTTest 支付手续费，所以这里通过 admin 的私钥创建 account
> const account = muta_sdk.Muta.accountFromPrivateKey('0x2b672bb959fa7a852d7259b129b65aee9c83b39f427d6f7bded1f58c4c9310c2')

> const as = new service.AssetService(client, account)

# 发行资产
> const MT = await as.write.create_asset({name: 'Muta Token', supply: 1000000000, symbol: 'MT', precision: 8, relayable: true})
{
  txHash: '0xdeb4c4cee67b0e65f9177fbeb2d2519dd8b82830fbf6c52c4b101ab94608789e',
  height: '0x000000000000005b',
  cyclesUsed: '0x000000000000f230',
  events: [
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0xcff1002107105460941f797828f468667aa1a2db","sender":"0xcff1002107105460941f797828f468667aa1a2db","recipient":"0xcff1002107105460941f797828f468667aa1a2db","value":1000,"memo":"pledge tx failure fee"}',
      service: 'asset'
    },
    {
      data: '{"id":"0x868e4ff082c7e029e403d63ba1d04da31889a88cff2f545736ad442cac14f82e","name":"Muta Token","symbol":"MT","supply":1000000000,"precision":8,"issuers":["0xcff1002107105460941f797828f468667aa1a2db"],"relayable":true}',
      service: 'asset'
    }
  ],
  stateRoot: '0x9a488eee80d8df6b3fee8f406e0ee2bb0e2bb5b48313f4af3ce9ba1f5da2acb9',
  response: {
    serviceName: 'asset',
    method: 'create_asset',
    response: {
      code: '0x0000000000000000',
      errorMessage: '',
      succeedData: [Object]
    }
  }
}

> const asset_id = MT.response.response.succeedData.id

# 发行者即为发交易的账户地址
> account.address
'0xcff1002107105460941f797828f468667aa1a2db'

# 查询发行者余额

> await client.queryService({serviceName: 'asset', method: 'get_balance', payload: JSON.stringify({asset_id: asset_id, user: account.address})})
{
  code: '0x0000000000000000',
  errorMessage: '',
  succeedData: '{"asset_id":"0x868e4ff082c7e029e403d63ba1d04da31889a88cff2f545736ad442cac14f82e","user":"0xcff1002107105460941f797828f468667aa1a2db","balance":1000000000}'
}

# 转账
> const to = accounts[1].address;

> await as.write.transfer({asset_id: asset_id, to, value: 100, memo:'Transfer MT to Bob'});
{
  txHash: '0xd36f68145b33fedcb2474fb0a75f79fcac89dcf025d6b8345918d63f3302b639',
  height: '0x000000000000007e',
  cyclesUsed: '0x000000000000f230',
  events: [
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0xcff1002107105460941f797828f468667aa1a2db","sender":"0xcff1002107105460941f797828f468667aa1a2db","recipient":"0xcff1002107105460941f797828f468667aa1a2db","value":1000,"memo":"pledge tx failure fee"}',
      service: 'asset'
    },
    {
      data: '{"asset_id":"0x868e4ff082c7e029e403d63ba1d04da31889a88cff2f545736ad442cac14f82e","from":"0xcff1002107105460941f797828f468667aa1a2db","to":"0x1ab16cd78f9b02f43fb6414b45cc6c7db6fdeaa0","value":100,"memo":"Transfer MT to Bob"}',
      service: 'asset'
    }
  ],
  stateRoot: '0x23a6693ee09766823e96eebcd647ec04a4574daca479d7e823eea1b7a50219e8',
  response: {
    serviceName: 'asset',
    method: 'transfer',
    response: { code: '0x0000000000000000', errorMessage: '', succeedData: {} }
  }
}

# 查看转账结果
> await client.queryService({ serviceName: 'asset', method: 'get_balance', payload: JSON.stringify({asset_id: asset_id, user: account.address})})
{
  code: '0x0000000000000000',
  errorMessage: '',
  succeedData: '{"asset_id":"0x868e4ff082c7e029e403d63ba1d04da31889a88cff2f545736ad442cac14f82e","user":"0xcff1002107105460941f797828f468667aa1a2db","balance":999999900}'
}

# 链上管理
> admin = muta_sdk.Muta.accountFromPrivateKey('0x2b672bb959fa7a852d7259b129b65aee9c83b39f427d6f7bded1f58c4c9310c2')

> admin.address
'0xcff1002107105460941f797828f468667aa1a2db'

> metadata_raw = await client.queryService({serviceName: 'metadata', method: 'get_metadata', payload: ''})
{
  code: '0x0000000000000000',
  errorMessage: '',
  succeedData: '{"chain_id":"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036","common_ref":"0x703873635a6b51513451","timeout_gap":99999,"cycles_limit":999999999999,"cycles_price":1,"interval":3000,"verifier_list":[{"bls_pub_key":"0x04188ef9488c19458a963cc57b567adde7db8f8b6bec392d5cb7b67b0abc1ed6cd966edc451f6ac2ef38079460eb965e890d1f576e4039a20467820237cda753f07a8b8febae1ec052190973a1bcf00690ea8fc0168b3fbbccd1c4e402eda5ef22","address":"0xf8389d774afdad8755ef8e629e5a154fddc6325a","propose_weight":1,"vote_weight":1}],"propose_ratio":15,"prevote_ratio":10,"precommit_ratio":10,"brake_ratio":7,"tx_num_limit":9000,"max_tx_size":10485760}'
}

> metadata = JSON.parse(metadata_raw.succeedData)
{
  chain_id: '0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036',
  common_ref: '0x703873635a6b51513451',
  timeout_gap: 99999,
  cycles_limit: 999999999999,
  cycles_price: 1,
  interval: 3000,
  verifier_list: [
    {
      bls_pub_key: '0x04188ef9488c19458a963cc57b567adde7db8f8b6bec392d5cb7b67b0abc1ed6cd966edc451f6ac2ef38079460eb965e890d1f576e4039a20467820237cda753f07a8b8febae1ec052190973a1bcf00690ea8fc0168b3fbbccd1c4e402eda5ef22',
      address: '0xf8389d774afdad8755ef8e629e5a154fddc6325a',
      propose_weight: 1,
      vote_weight: 1
    }
  ],
  propose_ratio: 15,
  prevote_ratio: 10,
  precommit_ratio: 10,
  brake_ratio: 7,
  tx_num_limit: 9000,
  max_tx_size: 10485760
}

# 构造交易修改 interval
> update_metadata_tx = await client.composeTransaction({ cyclesLimit: '0xffffff', method: 'update_interval', payload: { interval: metadata.interval - 1, }, serviceName: 'governance', sender: admin.address })
{
  chainId: '0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036',
  cyclesLimit: '0xffffff',
  cyclesPrice: '0xffff',
  nonce: '0x0677910f12241f85be84b854b8cc85655b349efe6b894032c363fcbcc351f207',
  timeout: '0x26fa',
  method: 'update_interval',
  payload: '{"interval":2999}',
  serviceName: 'governance',
  sender: '0xcff1002107105460941f797828f468667aa1a2db'
}

> hash = await client.sendTransaction(admin.signTransaction(update_metadata_tx))
'0xe45102c6cfdc66181174710138cdae1603a7906a608e8b75efe0fee391ffceab'

> await client.getReceipt(hash)
{
  txHash: '0xe45102c6cfdc66181174710138cdae1603a7906a608e8b75efe0fee391ffceab',
  height: '0x00000000000026ec',
  cyclesUsed: '0x000000000001e848',
  events: [
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0xcff1002107105460941f797828f468667aa1a2db","sender":"0xcff1002107105460941f797828f468667aa1a2db","recipient":"0xcff1002107105460941f797828f468667aa1a2db","value":1000,"memo":"pledge tx failure fee"}',
      service: 'asset'
    },
    { data: '{"interval":2999}', service: 'governance' },
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0xcff1002107105460941f797828f468667aa1a2db","sender":"0xcff1002107105460941f797828f468667aa1a2db","recipient":"0xcff1002107105460941f797828f468667aa1a2db","value":900,"memo":"collect tx fee"}',
      service: 'asset'
    }
  ],
  stateRoot: '0xd4391c7dfdcf3d20c7366c122efb565b43f854683d53826d4930c4715a127687',
  response: {
    serviceName: 'governance',
    method: 'update_interval',
    response: { code: '0x0000000000000000', errorMessage: '', succeedData: '' }
  }
}

# 再查一次可以发现 interval 已经改变了
> metadata_raw = await client.queryService({serviceName: 'metadata', method: 'get_metadata', payload: ''})
{
  code: '0x0000000000000000',
  errorMessage: '',
  succeedData: '{"chain_id":"0xb6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036","common_ref":"0x703873635a6b51513451","timeout_gap":99999,"cycles_limit":999999999999,"cycles_price":1,"interval":2999,"verifier_list":[{"bls_pub_key":"0x04188ef9488c19458a963cc57b567adde7db8f8b6bec392d5cb7b67b0abc1ed6cd966edc451f6ac2ef38079460eb965e890d1f576e4039a20467820237cda753f07a8b8febae1ec052190973a1bcf00690ea8fc0168b3fbbccd1c4e402eda5ef22","address":"0xf8389d774afdad8755ef8e629e5a154fddc6325a","propose_weight":1,"vote_weight":1}],"propose_ratio":15,"prevote_ratio":10,"precommit_ratio":10,"brake_ratio":7,"tx_num_limit":9000,"max_tx_size":10485760}'
}

# riscv service 演示

# 由于部署合约耗费 cycle 较大，调大 client 的默认 cycleslimit
> client.options.defaultCyclesLimit = '0xffffff'

> const fs = require('fs')

# 在 huobi-chain repo 根目录下，可以读取到下列示例合约
> const code = fs.readFileSync('tests/e2e/riscv_contracts/contract_test')

# 在 genesis.toml/riscv service 配置中我们可以看到只有 ’0x9cccacbb8a4b0353d42138613b2db72d6a661cf4‘ 这个地址被授予了部署合约的权限，所以下面的示例中我们将用这个账户进行合约的部署。

# 这个地址没有 `Huobi Test Token`，因为部署合约需要使用 `Huobi Test Token` 来支付手续费，所以我们先往这个地址转一点 `Huobi Token Test`

> await as.write.transfer({asset_id: '0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c', to: '0x9cccacbb8a4b0353d42138613b2db72d6a661cf4', value: 10000, memo:'Transfer MT to authAccount'})
{
  txHash: '0xfce5875f3c87fc9cf1a6303b2a61f705dd9815c44dc925cfa7e410fabe26e01e',
  height: '0x0000000000000399',
  cyclesUsed: '0x0000000000014438',
  events: [
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0xcff1002107105460941f797828f468667aa1a2db","sender":"0xcff1002107105460941f797828f468667aa1a2db","recipient":"0xcff1002107105460941f797828f468667aa1a2db","value":1000,"memo":"pledge tx failure fee"}',
      service: 'asset'
    },
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","from":"0xcff1002107105460941f797828f468667aa1a2db","to":"0x9cccacbb8a4b0353d42138613b2db72d6a661cf4","value":100,"memo":"Transfer MT to authAccount"}',
      service: 'asset'
    },
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0xcff1002107105460941f797828f468667aa1a2db","sender":"0xcff1002107105460941f797828f468667aa1a2db","recipient":"0xcff1002107105460941f797828f468667aa1a2db","value":900,"memo":"collect tx fee"}',
      service: 'asset'
    }
  ],
  stateRoot: '0xa113a49b67856a2a34bd677341eb8d9d95a3fc741880922118a8c178e26aeb6d',
  response: {
    serviceName: 'asset',
    method: 'transfer',
    response: { code: '0x0000000000000000', errorMessage: '', succeedData: {} }
  }
}

# 让我们来查一下有没有转成功
> await client.queryService({ serviceName: 'asset', method: 'get_balance', payload: JSON.stringify({asset_id: '0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c', user:'0x9cccacbb8a4b0353d42138613b2db72d6a661cf4'})})
{
  code: '0x0000000000000000',
  errorMessage: '',
  succeedData: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","user":"0x9cccacbb8a4b0353d42138613b2db72d6a661cf4","balance":10000}'
}

> auth = muta_sdk.Muta.accountFromPrivateKey('d6ef93ed5d27327fd10349a75d3b7a91aa5c1d0f42994be10c1cb0e357e722f5')

# 万事具备，现在来构造和发送部署合约的交易

> const tx = await client.composeTransaction({ method: 'deploy', payload: { intp_type: 'Binary', init_args: '', code: code.toString('hex') }, serviceName: 'riscv', sender: auth.address })

> txHash = await client.sendTransaction(auth.signTransaction(tx))
'0xea4f5f59047e5964e21ac70d4a4ce250c8c5f643fa74d623636de8e5c8c0abaa'

# 获取一下 receipt，确认下合约部署是否成功
> receipt = await client.getReceipt(txHash)
{
  txHash: '0x247c85cc354a5025d171981a7451336eea6cfeca51e24dee6a69032dffdc24b9',
  height: '0x00000000000006a6',
  cyclesUsed: '0x0000000000020170',
  events: [
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0x9cccacbb8a4b0353d42138613b2db72d6a661cf4","sender":"0x9cccacbb8a4b0353d42138613b2db72d6a661cf4","recipient":"0xcff1002107105460941f797828f468667aa1a2db","value":1000,"memo":"pledge tx failure fee"}',
      service: 'asset'
    },
    {
      data: '{"asset_id":"0xf56924db538e77bb5951eb5ff0d02b88983c49c45eea30e8ae3e7234b311436c","caller":"0x9cccacbb8a4b0353d42138613b2db72d6a661cf4","sender":"0xcff1002107105460941f797828f468667aa1a2db","recipient":"0x9cccacbb8a4b0353d42138613b2db72d6a661cf4","value":900,"memo":"collect tx fee"}',
      service: 'asset'
    }
  ],
  stateRoot: '0xa9c178cca25e3034e019552a97c6cf17acf3414537570dca91c45da2e677b2bd',
  response: {
    serviceName: 'riscv',
    method: 'deploy',
    response: {
      code: '0x0000000000000000',
      errorMessage: '',
      succeedData: '{"address":"0xf34ff2e82bc1e8c2b03826c6ae31daded4b1d0dc","init_ret":""}'
    }
  }
}

```
# How to Deploy POA Bridge Contracts

In order to deploy bridge contracts you must run `npm install` to install all dependencies. For more information, see the [project README](../README.md).

1. Compile the source contracts.
```
cd ..
npm run compile
```

2. Create a `.env` file.
```
cd deploy
cp .env.example .env
```

3. If necessary, deploy and configure a multi-sig wallet contract to manage the bridge contracts after deployment. We have not audited any wallets for security, but have used https://github.com/gnosis/MultiSigWallet/ with success.

4. Adjust the parameters in the `.env` file depending on the desired bridge mode. See below for comments related to each parameter.

5. Add funds to the deployment accounts in both the Home and Foreign networks.

6. Run `npm run deploy`.

## `ERC-TO-NATIVE` Bridge Mode Configuration Example.

This example of an `.env` file for the `erc-to-native` bridge mode includes comments describing each parameter.

```bash
# 桥的类型。定义要部署的合同集。
BRIDGE_MODE=ERC_TO_NATIVE

# 合约负责账户的私钥十六进制值
# 部署和初始配置。该帐户的余额必须包含
# 来自两个网络的资金。
DEPLOYMENT_ACCOUNT_PRIVATE_KEY=67..14
# 添加到特定部署配置交易的估计气体中的额外气体
# E.g. 如果预估gas返回100000且参数为0.2，
# 交易气体限额将为 (100000 + 100000 * 0.2) = 120000
DEPLOYMENT_GAS_LIMIT_EXTRA=0.2
# 在每个部署配置交易中设置的“gasPrice”参数
# Home network (in Wei).
HOME_DEPLOYMENT_GAS_PRICE=1000000000
# The "gasPrice" parameter set in every deployment/configuration transaction on
# Foreign network (in Wei).
FOREIGN_DEPLOYMENT_GAS_PRICE=5000000000
# 等待接收部署配置事务的超时限制。
GET_RECEIPT_INTERVAL_IN_MILLISECONDS=3000

# The RPC channel to a Home node able to handle deployment/configuration
# transactions.
HOME_RPC_URL=https://core.poa.network
# 具有更改桥接合同参数权限的家庭网络地址。为了额外的安全性，我们建议在此处使用多重签名钱包合约地址。
HOME_BRIDGE_OWNER=0x
# 具有更改桥接验证器合约参数权限的家庭网络地址。
HOME_VALIDATORS_OWNER=0x
# 具有升级桥接合约和桥接验证器合约权限的家庭网络地址。
HOME_UPGRADEABLE_ADMIN=0x
# 魏每日交易限额。一旦超过此限制，任何请求中继资产的交易都将失败。
HOME_DAILY_LIMIT=30000000000000000000000000
# Wei单笔交易的最大限额。如果单个交易试图中继超过此限制的资金，它将失败。 HOME_MAX_AMOUNT_PER_TX 必须小于 HOME_DAILY_LIMIT。
HOME_MAX_AMOUNT_PER_TX=150000000000000000000000
# Wei单笔交易的最低限额。如果交易试图中继低于此限制的资金，它将失败。这是防止干涸验证者帐户所必需的。
HOME_MIN_AMOUNT_PER_TX=50000000000000000
# 最终确定阈值。保证交易不会回滚的对应存款交易的区块后发出的区块数量。
HOME_REQUIRED_BLOCK_CONFIRMATIONS=1
# 用于发送家庭网络签名交易以确认存款或取款的默认汽油价格（以 Wei 为单位）。如果无法访问 Gas 价格预言机，则使用此价格。
HOME_GAS_PRICE=1000000000

# Home 网络上用于区块奖励计算的现有智能合约的地址。
BLOCK_REWARD_ADDRESS=0x

# The RPC channel to a Foreign node able to handle deployment/configuration
# transactions.
FOREIGN_RPC_URL=https://mainnet.infura.io
# Address on Foreign network with permissions to change parameters of the bridge contract.
# For extra security we recommended using a multi-sig wallet contract address here.
FOREIGN_BRIDGE_OWNER=0x
# Address on the Foreign network with permissions to change parameters of
# the bridge validator contract.
FOREIGN_VALIDATORS_OWNER=0x
# 具有升级桥接合约和桥接验证器合约权限的外部网络地址。
FOREIGN_UPGRADEABLE_ADMIN=0x
# wei 每日交易限额。在 Home 端用于检查桥验证器的操作。
FOREIGN_DAILY_LIMIT=15000000000000000000000000
# The maximum limit for one transaction in Wei. FOREIGN_MAX_AMOUNT_PER_TX must be
# less than FOREIGN_DAILY_LIMIT. Used on the Home side to check the bridge validator’s actions.
FOREIGN_MAX_AMOUNT_PER_TX=750000000000000000000000
# 在此模式下不使用，注释掉或删除该变量。
# FOREIGN_MIN_AMOUNT_PER_TX=
# 最终确定阈值。保证交易不会回滚的对应存款交易的区块后发出的区块数量。
FOREIGN_REQUIRED_BLOCK_CONFIRMATIONS=8
# 用于发送外国网络交易最终确定资产存款的默认 gas 价格（以 Wei 为单位）。如果无法访问 Gas 价格预言机，则使用此价格。
FOREIGN_GAS_PRICE=10000000000

# Foreign 网络中现有的 ERC20 兼容令牌的地址，用于交换到 Home 上的本地硬币。
ERC20_TOKEN_ADDRESS=0x

# 发送确认资产中继签名所需的最少验证器数量。桥的两侧预计会有相同数量的验证者。
REQUIRED_NUMBER_OF_VALIDATORS=1
# 验证者地址集。假设来自这些地址的签名是在 Home 端收集的。
# 相同的地址将在外国网络上使用，以确认最终协议已正确传输到外国网络。
VALIDATORS=0x


# 用于定义是否使用 RewardableValidators 合约并在 Home 网络上设置费用管理器合约的变量在此桥接模式下，Home 网络仅支持 BOTH_DIRECTIONS
HOME_REWARDABLE=false
# 用于定义是否使用 RewardableValidators 合约并在国外网络上设置费用管理合约的变量 此桥接模式不支持在国外网络上收取费用。
FOREIGN_REWARDABLE=false
# 用于定义 Home 网络是否为 POSDAO 的变量，奖励是否由 blockReward 合约分配给网络验证者或直接转移给桥接验证者。
# 支持的值为 BRIDGE_VALIDATORS_REWARD 和 POSDAO_REWARD
HOME_FEE_MANAGER_TYPE=BRIDGE_VALIDATORS_REWARD
# 应转移奖励的验证者帐户列表，以空格分隔，不带引号
# 仅当 HOME_REWARDABLE=BOTH_DIRECTIONS 时才有意义
VALIDATORS_REWARD_ACCOUNTS=0x 0x 0x

# 从本地网络到外国网络的每笔交易都要收取的费用
# Makes sense only when HOME_REWARDABLE=BOTH_DIRECTIONS
# e.g. 0.1% fee
HOME_TRANSACTIONS_FEE=0.001
# 从国外网络到国内网络的每笔交易都要收取的费用
# Makes sense only when HOME_REWARDABLE=BOTH_DIRECTIONS
# e.g. 0.1% fee
FOREIGN_TRANSACTIONS_FEE=0.001

# The api url of an explorer to verify all the deployed contracts in Home network. Supported explorers: Blockscout, etherscan
#HOME_EXPLORER_URL=https://blockscout.com/poa/core/api
# The api key of the explorer api, if required, used to verify all the deployed contracts in Home network.
#HOME_EXPLORER_API_KEY=
# The api url of an explorer to verify all the deployed contracts in Foreign network. Supported explorers: Blockscout, etherscan
#FOREIGN_EXPLORER_URL=https://api.etherscan.io/api
# The api key of the explorer api, if required, used to verify all the deployed contracts in Foreign network.
#FOREIGN_EXPLORER_API_KEY=
```

## `ARBITRARY-MESSAGE` Bridge Mode Configuration Example.

This example of an `.env` file for the `arbitrary-message` bridge mode includes comments describing each parameter.

```bash
# The type of bridge. Defines set of contracts to be deployed.
BRIDGE_MODE=ARBITRARY_MESSAGE

# The private key hex value of the account responsible for contracts
# deployments and initial configuration. The account's balance must contain
# funds from both networks.
DEPLOYMENT_ACCOUNT_PRIVATE_KEY=67..14
# Extra gas added to the estimated gas of a particular deployment/configuration transaction
# E.g. if estimated gas returns 100000 and the parameter is 0.2,
# the transaction gas limit will be (100000 + 100000 * 0.2) = 120000
DEPLOYMENT_GAS_LIMIT_EXTRA=0.2
# The "gasPrice" parameter set in every deployment/configuration transaction on
# Home network (in Wei).
HOME_DEPLOYMENT_GAS_PRICE=10000000000
# The "gasPrice" parameter set in every deployment/configuration transaction on
# Foreign network (in Wei).
FOREIGN_DEPLOYMENT_GAS_PRICE=10000000000
# The timeout limit to wait for receipt of the deployment/configuration
# transaction.
GET_RECEIPT_INTERVAL_IN_MILLISECONDS=3000

# The RPC channel to a Home node able to handle deployment/configuration
# transactions.
HOME_RPC_URL=https://poa.infura.io
# Address on Home network with permissions to change parameters of the bridge contract.
# For extra security we recommended using a multi-sig wallet contract address here.
HOME_BRIDGE_OWNER=0x
# Address on Home network with permissions to change parameters of bridge validator contract.
HOME_VALIDATORS_OWNER=0x
# Address on Home network with permissions to upgrade the bridge contract and the
# bridge validator contract.
HOME_UPGRADEABLE_ADMIN=0x
# The maximum value of gas for one call to be allowed for relaying.
HOME_MAX_AMOUNT_PER_TX=20000000
# The finalization threshold. The number of blocks issued after the block with
# the corresponding deposit transaction to guarantee the transaction will not be
# rolled back.
HOME_REQUIRED_BLOCK_CONFIRMATIONS=1
# The default gas price (in Wei) used to send Home Network signature
# transactions for deposit or withdrawal confirmations. This price is used if
# the Gas price oracle is unreachable.
HOME_GAS_PRICE=1000000000

# The RPC channel to a Foreign node able to handle deployment/configuration
# transactions.
FOREIGN_RPC_URL=https://mainnet.infura.io
# Address on Foreign network with permissions to change parameters of the bridge contract.
# For extra security we recommended using a multi-sig wallet contract address here.
FOREIGN_BRIDGE_OWNER=0x
# Address on Foreign network with permissions to change parameters of bridge validator contract.
FOREIGN_VALIDATORS_OWNER=0x
# Address on Foreign network with permissions to upgrade the bridge contract and the
# bridge validator contract.
FOREIGN_UPGRADEABLE_ADMIN=0x
# The maximum value of gas for one call to be allowed for relaying.
FOREIGN_MAX_AMOUNT_PER_TX=20000000
# The finalization threshold. The number of blocks issued after the block with
# the corresponding deposit transaction to guarantee the transaction will not be
# rolled back.
FOREIGN_REQUIRED_BLOCK_CONFIRMATIONS=8
# The default gas price (in Wei) used to send Foreign network transactions
# finalizing asset deposits. This price is used if the Gas price oracle is
# unreachable.
FOREIGN_GAS_PRICE=10000000000

# The minimum number of validators required to send their signatures confirming
# the relay of assets. The same number of validators is expected on both sides
# of the bridge.
REQUIRED_NUMBER_OF_VALIDATORS=1
# The set of validators' addresses. It is assumed that signatures from these
# addresses are collected on the Home side. The same addresses will be used on
# the Foreign network to confirm that the finalized agreement was transferred
# correctly to the Foreign network.
VALIDATORS=0x 0x 0x

# The api url of an explorer to verify all the deployed contracts in Home network. Supported explorers: Blockscout, etherscan
#HOME_EXPLORER_URL=https://blockscout.com/poa/core/api
# The api key of the explorer api, if required, used to verify all the deployed contracts in Home network.
#HOME_EXPLORER_API_KEY=
# The api url of an explorer to verify all the deployed contracts in Foreign network. Supported explorers: Blockscout, etherscan
#FOREIGN_EXPLORER_URL=https://api.etherscan.io/api
# The api key of the explorer api, if required, used to verify all the deployed contracts in Foreign network.
#FOREIGN_EXPLORER_API_KEY=
```

## `AMB-ERC-TO-ERC` Bridge Mode Configuration Example.

This example of an `.env` file for the `AMB-ERC-TO-ERC` bridge mode includes comments describing each parameter.

```bash
# The type of bridge. Defines set of contracts to be deployed.
BRIDGE_MODE=AMB_ERC_TO_ERC

# The private key hex value of the account responsible for contracts
# deployments and initial configuration. The account's balance must contain
# funds from both networks.
DEPLOYMENT_ACCOUNT_PRIVATE_KEY=67..14
# Extra gas added to the estimated gas of a particular deployment/configuration transaction
# E.g. if estimated gas returns 100000 and the parameter is 0.2,
# the transaction gas limit will be (100000 + 100000 * 0.2) = 120000
DEPLOYMENT_GAS_LIMIT_EXTRA=0.2
# The "gasPrice" parameter set in every deployment/configuration transaction on
# Home network (in Wei).
HOME_DEPLOYMENT_GAS_PRICE=10000000000
# The "gasPrice" parameter set in every deployment/configuration transaction on
# Foreign network (in Wei).
FOREIGN_DEPLOYMENT_GAS_PRICE=10000000000
# The timeout limit to wait for receipt of the deployment/configuration
# transaction.
GET_RECEIPT_INTERVAL_IN_MILLISECONDS=3000

# The name of the ERC677 token to be deployed on the Home network.
BRIDGEABLE_TOKEN_NAME=Your New Bridged Token
# The symbol name of the ERC677 token to be deployed on the Home network.
BRIDGEABLE_TOKEN_SYMBOL=TEST
# The number of supportable decimal digits after the "point" in the ERC677 token
# to be deployed on the Home network.
BRIDGEABLE_TOKEN_DECIMALS=18
# The flag defining whether to use ERC677BridgeTokenRewardable contract instead of
# ERC677BridgeToken on Home network.
DEPLOY_REWARDABLE_TOKEN=false
# The address of Staking contract used by ERC677BridgeTokenRewardable contract.
# Makes sense only when DEPLOY_REWARDABLE_TOKEN=true
#DPOS_STAKING_ADDRESS=0x
# The address of BlockReward contract used by ERC677BridgeTokenRewardable contract.
# Makes sense only when DEPLOY_REWARDABLE_TOKEN=true
#BLOCK_REWARD_ADDRESS=0x

# The RPC channel to a Home node able to handle deployment/configuration
# transactions.
HOME_RPC_URL=https://core.poa.network
# Address on Home network with permissions to change parameters of the bridge contract.
# For extra security we recommended using a multi-sig wallet contract address here.
HOME_BRIDGE_OWNER=0x
# Address on Home network with permissions to upgrade the bridge contract
HOME_UPGRADEABLE_ADMIN=0x
# The daily transaction limit in Wei. As soon as this limit is exceeded, any
# transaction which requests to relay assets will fail.
HOME_DAILY_LIMIT=30000000000000000000000000
# The maximum limit for one transaction in Wei. If a single transaction tries to
# relay funds exceeding this limit it will fail. HOME_MAX_AMOUNT_PER_TX must be
# less than HOME_DAILY_LIMIT.
HOME_MAX_AMOUNT_PER_TX=1500000000000000000000000
# The minimum limit for one transaction in Wei. If a transaction tries to relay
# funds below this limit it will fail. This is required to prevent dryout
# validator accounts.
HOME_MIN_AMOUNT_PER_TX=500000000000000000

# The RPC channel to a Foreign node able to handle deployment/configuration
# transactions.
FOREIGN_RPC_URL=https://mainnet.infura.io
# Address on Foreign network with permissions to change parameters of the bridge contract.
# For extra security we recommended using a multi-sig wallet contract address here.
FOREIGN_BRIDGE_OWNER=0x
# Address on Foreign network with permissions to upgrade the bridge contract and the
# bridge validator contract.
FOREIGN_UPGRADEABLE_ADMIN=0x
# The daily limit in Wei. As soon as this limit is exceeded, any transaction
# requesting to relay assets will fail.
FOREIGN_DAILY_LIMIT=15000000000000000000000000
# The maximum limit per one transaction in Wei. If a transaction tries to relay
# funds exceeding this limit it will fail. FOREIGN_MAX_AMOUNT_PER_TX must be less
# than FOREIGN_DAILY_LIMIT.
FOREIGN_MAX_AMOUNT_PER_TX=750000000000000000000000
# The minimum limit for one transaction in Wei. If a transaction tries to relay
# funds below this limit it will fail.
FOREIGN_MIN_AMOUNT_PER_TX=500000000000000000
# The address of the existing ERC20/ERC677 compatible token in the Foreign network to
# be exchanged to the ERC20/ERC677 token deployed on Home.
ERC20_TOKEN_ADDRESS=0x

# The address of the existing AMB bridge in the Home network that will be used to pass messages
# to the Foreign network.
HOME_AMB_BRIDGE=0x
# The address of the existing AMB bridge in the Foreign network that will be used to pass messages
# to the Home network.
FOREIGN_AMB_BRIDGE=0x
# The gas limit that will be used in the execution of the message passed to the mediator contract
# in the Foreign network.
HOME_MEDIATOR_REQUEST_GAS_LIMIT=2000000
# The gas limit that will be used in the execution of the message passed to the mediator contract
# in the Home network.
FOREIGN_MEDIATOR_REQUEST_GAS_LIMIT=2000000

# The api url of an explorer to verify all the deployed contracts in Home network. Supported explorers: Blockscout, etherscan
#HOME_EXPLORER_URL=https://blockscout.com/poa/core/api
# The api key of the explorer api, if required, used to verify all the deployed contracts in Home network.
#HOME_EXPLORER_API_KEY=
# The api url of an explorer to verify all the deployed contracts in Foreign network. Supported explorers: Blockscout, etherscan
#FOREIGN_EXPLORER_URL=https://api.etherscan.io/api
# The api key of the explorer api, if required, used to verify all the deployed contracts in Foreign network.
#FOREIGN_EXPLORER_API_KEY=
```

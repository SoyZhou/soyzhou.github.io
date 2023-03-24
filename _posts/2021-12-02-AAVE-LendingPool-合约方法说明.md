---
title: AAVE LendingPool(V2) 合约方法说明
date: 2021-12-02 11:53:58
tags: ['blockchain','defi','smart contract','tech']
---

## 0 合约介绍

`LendingPool` 合约是 `aave protocol` 的主要合约。所有面向用户的方法，都可以使用智能合约或者web3调用。

主合约源代码地址：[点击查看](https://github.com/aave/protocol-v2/blob/ice/mainnet-deployment-03-12-2020/contracts/protocol/lendingpool/LendingPool.sol)

> 方法 deposit, borrow, withdraw and repay 只支持 ERC20，如果需要调用原生币（ETH、MATIC等）需要使用 [WETHGateway](https://docs.aave.com/developers/the-core-protocol/weth-gateway)。

## 1 合约方法

> Tips:
> 1. 所有的数量都不带精度


> 一些通用参数
> - asset: 币种地址
> - referralCode: 自己调用就写0
> - onBehalfOf: 行为地址。合约调用填入 `msg.sender`


---

### deposit - 存款

`function deposit(address asset, uint256 amount, address onBehalfOf, uint16 referralCode)`

amount: 充入的数量


> 注意⚠️：调用前需要先调用该币种的 `approve` 方法，给予改合约足够的额度。谁调用 LendingPool 谁给额度。如果是你调用你的合约去调用 LendingPool，那么就该你的合约 approve 给 LendingPool。

**合约调用代码**
```solidity
// 合约是需要先有资产余额，才能转入
function deposit(address asset, uint256 amount) public {
  IERC20 _asset = IERC20(asset);

  // 需要先将资产从 msg.sender 转入到合约中
  _asset.transferFrom(msg.sender, address(this), amount)

  // 该合约也需要给 LendingPool approve 对应的额度
  _asset.approve(address(_lendingPool), amount);

  // deposit，但是对应归属记录到 msg.sender 上
  _lendingPool.deposit(asset, amount, msg.sender, 0);
}
```

---

### withdraw - 取款

`function withdraw(address asset, uint256 amount, address to)`

amount: 提币数量。使用 `type(uint).max` 提出所有余额。

to: 接收提币的地址


**合约调用代码**
```solidity
// 需要先将 aToken 转入合约，才能提款
function withdraw(address asset, uint256 amount) public {
  // 获得 aToken 地址
  (address aTokenAddress,,) = _dataProvider.getReserveTokensAddresses(asset);

  // 先把 aToken 转到合约中来
  IERC20(aTokenAddress).transferFrom(msg.sender, address(this), amount);

  // 再取出 asset
  _lendingPool.withdraw(asset, amount, address(this));
}
```

---

### borrow - 借款

`function borrow(address asset, uint256 amount, uint256 interestRateMode, uint16 referralCode, address onBehalfOf)`

amount: 想要借出的数量

interestRateMode: 利率模式。1，固定利率。2，浮动利率。

**合约调用代码**
```solidity
// 借款，借到的余额会打到这个合约里面，需要 vToken 或者 sToken 先 approveDelegation 给该合约
function borrow(address asset, uint256 amount) private {
  // 选择 2 浮动利率
  _lendingPool.borrow(asset, amount, 2, 0, msg.sender);
}
```

---

### repay - 还款

`function repay(address asset, uint256 amount, uint256 rateMode, address onBehalfOf)`

amount: 还款数量。只有自己调用时才可以使用`uint(-1)`进行全部还款。第三方调用时，建议还款数量略大于目前的借款数量。

rateMode: 借款利率模式。1，固定利率。2，浮动利率。


**合约调用代码**
```solidity
// 合约是需要先有资产余额，才能还款（如果还款没有用完，余额会在合约里面，需要自己转出来，或者 deposit 到 LendingPool 里面）
function deposit(address asset, uint256 amount) public {
  IERC20 _asset = IERC20(asset);

  // 需要先将资产从 msg.sender 转入到合约中
  _asset.transferFrom(msg.sender, address(this), amount)

  // 该合约也需要给 LendingPool approve 对应的额度
  _asset.approve(address(_lendingPool), amount);

  // repay msg.sender 的借款
  _lendingPool.repay(asset, amount, 2, msg.sender);
}
```

---

### swapBorrowRateMode - 修改借款模式

`function swapBorrowRateMode(address asset, uint256 rateMode)`

rateMode: 借款利率模式。1，固定利率。2，浮动利率。

---

### setUserUseReserveAsCollateral - 设置资产是否能作为抵押物

`function setUserUseReserveAsCollateral(address asset, bool useAsCollateral)`

useAsCollateral: `true`可以作为抵押物

---

### liquidationCall

`function liquidationCall(address collateral, address debt, address user, uint256 debtToCover, bool receiveAToken)`


---

### flashLoan - 闪贷

`function flashLoan(address receiverAddress, address[] calldata assets, uint256[] calldata amounts, uint256[] modes, address onBehalfOf, bytes calldata params, uint16 referralCode)`

receiverAddress: 接收贷款的地址

assets: 资产地址列表，注意是列表。

amounts: 对应的资产数量，注意是列表。

modes: 如果没有还款成功，要选择的贷款方式。0，不要贷款直接revert。1，固定利率贷款。2，浮动利率贷款。注意是列表。（建议选0）

onBehalfOf: 如果贷款模式不是0，这笔贷款就会记在这个地址上。需确认 onBehalfOf 的地址已经 approve 了足够的金额给 `msg.sender`

params: 要传回 `receiverAddress` 合约地址的参数的字节编码。

> 使用闪贷的合约中需要存在 `executeOperation` 方法，在合约接收到贷款时，会回调改方法，在该方法的最后需要 `approve` 给 `LendingPool` 合约对应需要还款的数量，结束时会自动收回贷款及贷款手续费。

## 2 合约地址

[ETH contracts](https://docs.aave.com/developers/deployed-contracts/deployed-contracts)

[Matic contracts](https://docs.aave.com/developers/deployed-contracts/matic-polygon-market)

[Avalanche contracts](https://docs.aave.com/developers/deployed-contracts/avalanche-market)
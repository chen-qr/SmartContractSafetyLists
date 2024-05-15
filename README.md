# SmartContractSafetyLists
智能合约安全问题整理


## 1. 重入攻击（Reentrancy Attack）

智能合约的重入攻击是攻击者通过反复调用合约函数，利用合约在更新状态前进行外部调用的漏洞，非法获取资金或进行其他操作。

### 外部调用：为什么重入攻击发生在外部调用时？

因为，外部调用，会让当前合约A失去执行权，让外部合约B获得执行权。获得执行权后外部合约B就可以做坏事。

- 当合约A调用合约B的函数时，例如发送以太币给一个外部地址，这个调用实际上是一个外部调用。
- 外部合约B（或地址）在接收到调用时，获得了执行控制权。此时，外部合约B可以执行它自己的代码。
- 如果外部合约B的代码在处理外部调用的过程中，再次调用了原始合约A的一个函数，而这个函数可能还未完成之前的**状态更新**操作，就会导致重入攻击。

### 执行控制权转移：重入攻击的根本原因

EVM是**单线程**执行模型，该模型决定了智能合约的执行控制权需要在EVM内发生转移。

- 调用栈：合约不管是内部调用还是外部调用，都会产生一个调用栈。每次调用都会在调用栈的基础上新增一层，直达达到最大的调用深度。
- 内部调用：合约内部函数之间的调用，也会产生调用栈，但执行控制权不会离开合约。
- 外部调用：一个合约调用另一个合约的函数，或者通过call、delegatecall、staticcall等方法进行调用，会把执行控制权转移给外部合约。

### [重入攻击解决办法](./readmes/1_重入攻击解决办法.md)

## 2. 权限控制问题（Access Control Issues）

合约中重要敏感的操作，没有设置权限验证，被攻击者调用。

解决方案：使用权限控制模式，如OpenZeppelin的Ownable协议或AccessControl协议。

Ownable协议适合简单权限控制；AccessControl适合多角色和权限的管理。

## 3. 短地址攻击（Short Address Attack）

利用以太坊合约中参数解析的漏洞，攻击者通过发送短地址来引起合约解析错误，进而执行恶意行为。

以太坊地址是固定长度的20字节（40个十六进制字符），而以太坊智能合约中的输入数据是按照32字节（256位）对齐的。当一个交易的输入数据长度不足32字节时，EVM会在末尾填充零来对齐数据。攻击者可以利用这一点，在构造交易数据时故意省略地址的部分字符，使得合约在解析地址时出现错误。

- 构造不完整地址：攻击者构造一个包含不完整地址的交易数据。例如，将地址0x1234567890abcdef1234567890abcdef12345678缩短为0x1234567890abcdef12345678（省略8个字符）。
- 填充零字节：由于EVM会自动填充零字节，省略的部分将被零填充，变成0x1234567890abcdef123456780000000000000000。
- 参数解析错误：智能合约在解析这个输入数据时，将把地址解释为0x1234567890abcdef123456780000000000000000，而实际上这是一个无效的地址，但在合约逻辑中可能被误认为是有效的。

解决方案: 使用OpenZeppelin的RC20标准提供的地址转账操作。

## 4. 整数溢出和下溢（Integer Overflow and Underflow）

当算术操作超过数据类型的最大或最小值时，会导致意外的行为。

解决方案：使用SafeMath库来进行安全的算术操作。
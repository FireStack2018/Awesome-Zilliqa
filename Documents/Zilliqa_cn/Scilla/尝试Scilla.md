# 尝试Scilla

虽然Scilla还在积极开发中，但是现在就可以尝试一下用Scilla语言开发。

## 区块链IDE

了解Scilla的最好办法是用一下[Scilla Blockchain IDE](https://wallet-scilla.zilliqa.com/)。IDE通过[testnet钱包](https://wallet-scilla.zilliqa.com/)和[区块浏览器](https://explorer-scilla.zilliqa.com/)连接到Zilliqa区块链，因此具有在真实区块链环境中测试Scilla合约所需的几乎所有功能。

要使用[Scilla Blockchain IDE](https://wallet-scilla.zilliqa.com/)，用户必须持有Testnet ZIL（拥有token才能使用Zilliqa区块链的基础功能）。这些token将定期免费分发。测试网络的ZIL tokens需要用来支付gas费用，才能部署和运行智能合约。

从非合约账户到非合约账户的每笔普通付款需花费1个单位的gas。每份合约的创建需花费50个单位的gas，而每个从非合约账户到合约账户的交易将花费10个单位的天然气。从合约账户到另一个合约账户的任何信息调用或其他行为也将花费10个单位的gas。

举个链式调用的例子：某位用户声称Alice调用了合约`C_1`，而该合约又调用另一个合约`C_2`，则此次合约调用总共花费20个单位的gas。

要试用Blockchain IDE，请点击进入[Zilliqa testnet钱包](https://wallet-scilla.zilliqa.com/)。

## 解释器IDE

[Scilla Interpreter IDE](https://ide.zilliqa.com/)是一个简单的开发环境，开发者可以用它来进行Scilla编码和测试。而[Scilla Blockchain IDE](https://wallet-scilla.zilliqa.com/)是一个用来测试Scilla合约的独立环境。因为它在后端运行Scilla解释器，并不连接到任何区块链网络，所以它不保留任何持久化状态数据，并且获取不了任何区块链上的参数，例如当前块编号。

因此，合约撰写者或调用者必须模拟输入某些参数，例如当前合约和区块链状态等，并传递给解释器。请参阅[解释器接口](http://scilla.readthedocs.io/en/latest/interface.html#interface-label)以了解要传递给解释器的输入参数格式。

用户无需持有测试网络内的ZIL即可使用此IDE，请查阅[Scilla Interpreter IDE](https://ide.zilliqa.com/)开始试用。

## 合约示例

以上两个IDE都内含以Scilla编写的智能合约示例：

- **HelloWorld**：这是一个简单的合约，允许指定帐户`owner`通过`setHello(msg: String)`设置一条欢迎消息 。合约还提供了`getHello()` 方法，可返回带有欢迎消息的帐户。

- **众筹**：众筹实施了一个kickstarter活动，用户可使用`Donate()`向相关活动的合约捐款。如果众筹成功（在指定时间内募集够目标资金），募集资金将通过 `GetFunds()`发送到一个预设帐户`owner`。如果众筹失败，那么贡献者可以通过`ClaimBack()`收回他们的捐款。

- **Zil-game**：这是一款双人游戏，其目标是找到与给定的SHA256 digest最接近的原像（`puzzle`）。详细信息如下：给定一个digest *d*，及两个值*x*和*y*。如果在某个距离函数中，`Distance(SHA-256(x), d) < Distance(SHA-256(y), d)`，则*x*被认为是一个比*y*更接近原像的*d*。

  游戏分两个阶段进行。在第一阶段，玩家使用`Play(guess: Hash)`提交他们的哈希值，即SHA-256（x）和SHA-256（y）。一旦玩家一提交了她的哈希值，玩家二就必须在规定的时间内提交她的哈希值。如果玩家二没有在规定的时间内提交她的哈希值，那么玩家一会成为获胜者。在第二阶段，玩家需要使用`ClaimReward(solution: Int128)`提交相应的值`x`或`y` 。提交了最接近原像的玩家将成为获胜者并赢得奖励。如果没有玩家参与游戏，合约还提供了`Withdraw ()`来回收资金，并将资金发送给指定`owner`。

- **FungibleToken**：模仿ERC20风格的可替代性token标准的可替代性token合约。

- **OpenAuction**：一种简单的公开拍卖合约，投标人通过`Bid ()`进行投标。成交金额将转到预设帐户。未中标的投标人通过`Withdraw()`收回投标金额。拍卖组织者通过`AuctionEnd()`来宣布成交价。


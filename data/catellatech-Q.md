# QA report for Centrifuge contest by Catellatech

## [01] Unsafe way to make a new `LiquidityPool` in the contract Factory.sol
In the `Factory::newLiquidityPool()` function to create a new `LiquidityPool` the team used the `create` method, and its more secure and efficient deploy such contracts via `create2` with salt that includes msg.sender. 

### The function looks like:
```solidity
LiquidityPool liquidityPool = new LiquidityPool(poolId, trancheId, currency, trancheToken, investmentManager);
```
and there is 2 more instances of this in the `Factory.sol` contract.

## [02] In `InvesmentManager.sol` contract the team repeat the same in the functions
for example in the contract is repeated `address liquidityPool = msg.sender;` six times, this is redundant and should be write like a modifier function. And the same for the `PoolManager` contract, the `require(address(trancheToken) != address(0), "PoolManager/unknown-token");` is repeated five times.


## [03] Functions overloading in the `ERC20.sol` contract
Having multiple functions with the same name in a smart contract can be dangerous or not a good practice for several reasons:

- Confusion: If there are several functions with the same name, it can be confusing for developers and users who are interacting with the smart contract. This can lead to errors and misunderstandings in the use of the contract.

- Security vulnerabilities: If multiple functions are defined with the same name, attackers can attempt to exploit this vulnerability to access or modify data or functionalities of the smart contract.

- Network overload: If there are multiple functions with the same name, there may be an impact on the efficiency and speed of the contract, as the network may be confused in trying to determine which function should be executed.

See more about this on [Solidity Docs.](https://docs.soliditylang.org/en/v0.8.19/contracts.html#function-overloading)

### Function where this happens
```solidity
216: function permit(address owner, address spender, uint256 value, uint256 deadline, bytes memory signature) public

239: function permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
        external
```

## [04] Remove non-standard increaseAllowance and decreaseAllowance from `ERC20.sol`
The team should remove non-standard increaseAllowance and decreaseAllowance from `ERC20.sol`.

## [05] Solidity version >=0.8.20 may not work on other chains due to `PUSH0`
The compiler for Solidity >=0.8.20 switches the default target EVM version to Shanghai, which includes the new PUSH0 op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier EVM version. While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

## [06] In `LiquidityPool::mint()` the comment should be removed from the code
Remove the following comment from the code`// require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");` .

## [07] In `Factory::constructor()` check this important input
The team should check this important input, the address `_root` must be different from address(0).

## [08] Use descriptive names for better readability 
In some cases in the scope some names don't describe the purpose of what supposed to do. for example in `LiquidityPool::file()`. 

The function looks like:
```solidity
  function file(bytes32 what, address data) public auth {
        if (what == "investmentManager") investmentManager = InvestmentManagerLike(data);
        else revert("LiquidityPool/file-unrecognized-param");
        emit File(what, data);
    }
```

## [09] take advantage of custom error's return value property
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the `()` sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

Consider take advantage of custom error's to future debbugs.

## [10] Inconsistent Use of NatSpec
NatSpec is a boon to all Solidity developers. Not only does it provide a structure for developers to document their code within the code itself, it encourages the practice of documenting code. When future developers read code documented with NatSpec, they are able to increase their capacity to understand, upgrade, and fix code. Without code documented with NatSpec, that capacity is hindered.

The Centrifuge codebase does have a high level of documentation with NatSpec. However there are numerous instances of functions missing NatSpec.

### Possible Risks
1. Lack of understanding of code without adequate documentation.
2. Difficulty in understanding, upgrading, and fixing code without documentation.

### Recommended Mitigation Steps
- Practice consistent use of NatSpec on all parts of the project codebase.
- Include the need for NatSpec comments for code reviews to pass.


Using old versions of 3rd-party libraries in the build process can introduce vulnerabilities to the protocol code.

- Latest is 4.9.3 (as of July 28, 2023), version used in project is 4.9.0

### Risks
- Older versions of OpenZeppelin contracts might not have fixes for known security vulnerabilities.
- Older versions might contain features or functionality that have been deprecated in recent versions.
- Newer versions often come with new features, optimizations, and improvements that are not available in older versions.

### Recommendation
Review the latest version of the OpenZeppelin contracts and upgrade.

## [11] Inconsistent coding style
An inconsistent coding style can be found throughout the codebase. Some examples include:
Some of the **parameter names** in certain functions lack the underscores as seen in other cases. To enhance code `readability` and adhere to the Solidity style guide standards, it is recommended to incorporate underscores in the parameter names. **This practice contributes to maintaining a consistent and organized codebase.**

To improve the overall consistency and readability of the codebase, consider adhering to a more consistent coding style by following a specific style guide.

## [12] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [13] Crucial information is missing on important functions in all contracts
In the constructor functions it is not specified in the documentation if the admin/roles will be an EOA or a contract. Consider improving the docstrings to reflect the exact intended behaviour, and using `Address.isContract` function from OpenZeppelin’s library to detect if an address is effectively a contract.

## [14] Zero as a function argument should have a descriptive meaning
Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.
For example in `InvestmentManager::_deposit()` is used `require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");`.

## [16] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.
For example:

```Solidity 
mapping(address account => uint256 balance); 
```
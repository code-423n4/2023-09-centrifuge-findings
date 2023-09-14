 
# Gas Optimizations Report

## Summary

[G-01] Cache storage variables to stack variables to save gas (04 Instances)
[G-02] Increase number of optimisation runs (02 Instances)
[G-03] Use calldata instead of memory for function parameters (08 Instances)
[GAS 04] Using private rather than public for immutable, saves gas (16 Instances)
[G 05] abi.encode() is less efficient than abi.encodepacked()(16 Instances)
[G 06] String literals passed to abi.encode()/abi.encodePacked() should not be split by commas (01 Instances)
[G-07] Use assembly to check for address(0) (21 Instances)
[G-08] Structs can be packed into fewer storage slots (02 Instances)


# [G-01] Cache storage variables to stack variables to save gas (04 Instances)
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.
In general, if a state variable is read more than once, caching its value to a local variable and reusing it will save gas since a storage read spends more gas than a memory write plus a memory read.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L325
2.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L326
3.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L138
4.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L143


# [G-02] Increase number of optimisation runs (02 Instances)
Optimisation runs are should be changes to 1000 instead of 200. This helps in removing unused opcodes which are the main element of gas consumption.
The gas of contract deployment will be comparably high but will cost less gas when functions are executed.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/foundry.toml#L23
2.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/foundry.toml#L34


# [G-03] Use calldata instead of memory for function parameters (08 Instances)

Using calldata in external function does not require data to be stored, which reduced the process time as compared to memory. This in return saves gas during calling the data.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L62
2.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L84
3.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L85
4.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L83
5.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L47
6.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L48
7.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L55
8.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L56


# [G 04] Using private rather than public for immutable, saves gas (16 Instances)
Private over public saves around 21,000 for immutable in gas during deployment.
Gas consumption also increases with number of variable created with public visibility for immutable.
Link to the code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L19
2.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/PauseAdmin.sol#L11
3.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/DelayedAdmin.sol#L13
4.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/routers/axelar/Router.sol#L29
5.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L55
6.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L56
7.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L62
8.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L67
9.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L75
10.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L76
11.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L81
12.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L82
13.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L83
14.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L22
15.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L30
16.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L85


# [G 05] abi.encode() is less efficient than abi.encodepacked()(16 Instances)
Refer to this [link]( https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison).

Link to the Code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L94
2.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L226
3.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L242
4.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L80
5.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L99
6.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L118
7.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L152
8.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L210
9.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L242
10.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L274
11.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L323
12.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L381
13.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L701
14.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L713
15.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L817
16.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L841


# [G 06] String literals passed to abi.encode()/abi.encodePacked() should not be split by commas (01 Instances)

String literals can be split into multiple parts and still be considered as a single string literal.
EACH new comma costs 21 gas due to stack operations and separate MSTOREs.
	
Link to the Code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L227


# [G-07] Use assembly to check for address(0) (21 Instances)
	
Link to the Code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L229
2.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L245
3.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L266
4.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L288
5.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L305
6.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L656
7.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L143
8.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L157
9.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L184
10.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L221
11.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L229
12.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L241
13.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L259
14.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L270
15.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L282
16.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L309
17.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L313
18.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L92
19.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L107
20.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L163
21.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L218


# [G-08] Structs can be packed into fewer storage slots (02 Instances)
Structs with proper slot allocation and similar data types when clustered together are more efficient.	
	
Link to the Code:
1.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L61
2.	https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L53

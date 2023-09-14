# Auther: Krace


# Approach

- Read the Readme.md to get an initial understanding of the project.
  - This is a Liquidity Pool Contract. Because I didn't participate in the competition for the first time, I didn't spend too much time reading the documentation, but focused on the code implementation
- Clone code and set up the test environment 
- Read the bot-report
- Analyzing the smart contract's logic 
  - Focus on Common Vulnerability Patterns: reentrancy attacks, integer overflow/underflow, and unauthorized access to sensitive functions or data.
  - Examined the use of external dependencies, ensuring that they were appropriately implemented and that their security implications were understood and accounted for.
  - Reviewed the contract's permissions and access control mechanisms, verifying that only authorized users had access to critical functions and data.
  - Simulated various scenarios, such as edge cases and unexpected inputs, to assess how the contract handled exceptional situations.


# Codebase quality analysis

## [LiquidityPool.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol)
### Analysis
I think this is the core of the contract, but after auditing I found that the main logic is implemented in `InvestmentManager`. 
This contract mainly provides the interfaces to interact with.


## [InvestmentManager.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol)
### Analysis
This contract is the implementation of `withdraw`, `deposit` and `redeem`, also interactives with `gateway` to handle the order.
### Problem
There are several uses of `mulDiv` in this contract, which may round down to zero due to precision loss.

## [PoolManager](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol)
### Analysis
This contract is responsible for managing the pools, including adding pools, currency and trancheToken. Only gateway could add pools and anyone could deploy the tranche and liquidityPool.
### Problem
The main problem is that the pool, curreny and tranche can only be added and can not be removed or modified. This might not be convenient to manage in certain situations.


## [Factory](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol)
### Analysis
In charge of creating new LiquidityPool and TrancheToken.


## [Root](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Root.sol)
### Analysis
Core contract that is a ward on all other deployed contracts
### Problem
Centralization risk


## [Gateway](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol)
### Analysis
Parsing incoming and outgoing messages. Incoming messages will be parsed and then sent to the corresponding Contract to handle. Outgoing message will be sent to outgoingRouter.




# Centralization && System risks
The authentication of this project mainly relied on the `Auth` contract, which maintains a map `wards` to store the permissions. An authenticated user could easily add anyone to the `wards` without any checks. In other words, all authorizations are on the same level. If both "a" and "b" as authorized users further authorize other users, "a" can revoke "b"'s original authorization without needing "b's" permission.
Meanwhile, the `Root` contract owns all the other contracts, it seems to be too powerful.
I think this kind of permission control is too centralized and fragile. Once one of the authorized users is abused, the entire contract is at risk. The advanced permissions of the contract should be managed through a decentralized voting process by a trusted community. Granting and revoking permissions should also have robust validation processes in place.



# Time spent
15 hours

### Time spent:
15 hours
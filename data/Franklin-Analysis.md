## Audit Details

Project Name: Centrifuge

Client Name: Centrifuge

Auditor: Franklin

Languages: Solidity

Date: September 14th 2023

**Audit Goals:** The focus of the audit was to verify that the smart contract system is secure, resilient and working according to its specifications. The audit activities can be grouped in the following

**Security:** Identifying security related issues within each contract and within the system of the contracts. 
**Sound Architecture:**  Evaluation of the architecture of this system through the lens of established smart contract best practices and general software best practices
**Code correctness and Quality:**  A full review of the contract source code. The primary area of focus includes:

Correctness,
Readability,
Sections of code with high complexity,
Quantity and quality of test coverage,

**Documentation**
The following documentation(s) was available to wardens and provides the necessary context for this report

https://docs.centrifuge.io/getting-started/securitization/ 



**Key Oberservations**

The coding style were heavily inspired by MarkerDAO’s coding style. No upgradeable proxies but rather using contract migrations, and as few dependencies as possible. Authentication uses the ward pattern, in which addresses can be relied or denied to get access. Centrifuge documentation were thorough and well written. The code is written defensively with frequent assert statements to revert calls with Invalid inputs


 
**Test Suite and Code Coverage**

On running code coverage, the following results were outputted: 

| File                                  | % Lines           | % Statements      | % Branches       | % Funcs          |
|---------------------------------------|-------------------|-------------------|------------------|------------------|
| script/Axelar.s.sol            | 0.00% (0/32)      | 0.00% (0/35)      | 0.00% (0/2)      | 0.00% (0/2)      |
| script/Deployer.sol           | 0.00% (0/40)      | 0.00% (0/42)      | 100.00% (0/0)    | 0.00% (0/4)      |
| script/Permissionless.s.sol | 0.00% (0/20)      | 0.00% (0/21)      | 0.00% (0/2)      | 0.00% (0/2)      |
| src/Escrow.sol                   | 100.00% (2/2)     | 100.00% (2/2)     | 100.00% (0/0)    | 100.00% (1/1)    |
| src/InvestmentManager.sol  | 97.35% (184/189)  | 96.47% (246/255)  | 61.11% (55/90)  |97.62% (41/42)   |
| src/LiquidityPool.sol         | 100.00% (65/65)   | 100.00% (86/86)   | 100.00% (4/4)    | 100.00% (38/38)  |
| src/PoolManager.sol             | 95.96% (95/99)    | 94.59% (105/111)  | 74.07% (40/54)   | 94.12% (16/17)   |
| src/Root.sol                          | 100.00% (22/22)   | 100.00% (22/22)   | 100.00% (8/8)    | 100.00% (8/8)    |
| src/UserEscrow.sol               | 100.00% (8/8)     | 100.00% (8/8)     | 100.00% (4/4)    | 100.00% (2/2)    |
| src/admins/DelayedAdmin.sol | 100.00% (4/4)  | 100.00% (4/4)     | 100.00% (0/0)    | 100.00% (4/4)    |
| src/admins/PauseAdmin.sol  | 100.00% (5/5)     | 100.00% (5/5)     | 100.00% (0/0)    | 100.00% (3/3)    |
| src/gateway/Gateway.sol     | 93.42% (71/76)    | 91.86% (79/86)    | 76.47% (26/34)   | 100.00% (17/17)  |
| src/gateway/Messages.sol   | 59.26% (96/162)   | 59.77% (153/256)  | 16.67% (1/6)     | 55.00% (44/80)   |
| src/gateway/routers/axelar/Router.sol | 100.00% (8/8) 100.00% (9/9)  | 100.00% (4/4)  |100.00% (3/3)    |
| src/gateway/routers/xcm/Router.sol | 0.00% (0/38)  | 0.00% (0/46)   | 0.00% (0/20)     | 0.00% (0/9)      |
| src/token/ERC20.sol              | 97.40% (75/77)    | 96.51% (83/86)    | 95.00% (38/40)   | 93.33% (14/15)   |
| src/token/RestrictionManager.sol |100.00% (15/15)  | 100.00% (17/17) | 80.00% (8/10)|100.00% (6/6)    |
| src/token/Tranche.sol                 | 100.00% (18/18)   | 100.00% (31/31)   | 100.00% (4/4)    | 75.00% (9/12)    |
| src/util/Auth.sol                     | 100.00% (4/4)     | 100.00% (4/4)     | 100.00% (0/0)    | 100.00% (2/2)    |
| src/util/BytesLib.sol                | 0.00% (0/28)      | 0.00% (0/28)      | 0.00% (0/16)     | 0.00% (0/7)      |
| src/util/Context.sol                 | 100.00% (1/1)     | 100.00% (1/1)     | 100.00% (0/0)    | 100.00% (1/1)    |
| src/util/Factory.sol              | 100.00% (24/24)   | 100.00% (37/37)   | 100.00% (0/0)    | 100.00% (3/3)    |
| src/util/MathLib.sol                | 0.00% (0/30)      | 0.00% (0/38)      | 0.00% (0/6)      | 0.00% (0/3)      |
| src/util/SafeTransferLib.sol    | 100.00% (7/7)     | 100.00% (9/9)     | 66.67% (4/6)     | 100.00% (3/3)    |
| test/TestSetup.t.sol                  | 0.00% (0/49)      | 0.00% (0/72)      | 0.00% (0/6)      | 0.00% (0/9)      |
| test/mock/AxelarGatewayMock.sol| 100.00% (4/4)  | 100.00% (4/4)  | 100.00% (0/0) |100.00% (2/2)    |
| test/mock/GatewayMock.sol | 2.13% (1/47)      | 2.13% (1/47)      | 100.00% (0/0)    | 9.09% (1/11)     |
| test/mock/Mock.sol               | 25.00% (1/4)      | 25.00% (1/4)      | 100.00% (0/0)    | 25.00% (1/4)     |
| Total                              | 65.86% (710/1078) | 66.40% (907/1366) | 62.03% (196/316) | 70.65% (219/310) |


Recommendations

Is great to see that some of functions in some contracts have a 100% coverage. Nonetheless, To improve the code coverage and ensure better testing, Consider writing additional test cases to cover untested parts of the code, especially areas with conditional logic (branches). This will help increase the overall confidence in the correctness and reliability of the smart contracts.
Additionally, it's essential to ensure that tests are comprehensive and cover various scenarios, including edge cases and error conditions, to provide robust testing coverage.

**Overview**

Centrifuge is the institutional platform for credit onchain. Centrifuge’s multi-chain strategy allows investors to access native RWA yields on the network of their choice.
Centrifuge works based on a hub-and-spoke model. RWA pools are managed by borrowers on Centrifuge Chain, an application-specific blockchain built purposely for managing real world assets.




 
**Issue Overview**

 The following are some of the issues discovered during the audit. 

1) Missing events when members are updated and when trancheMetaData are updated in Restriction Manager and Pool Manager.sol respectively. 

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L214 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L57 

Recommended Mitigation Steps
Consider Adding an event emitter when this variables are updated. This will help notify users about the correct name and symbol of the trancheToken and also keep note of the latest updated members. 


2) Consider clear naming convention of parameters to avoid confusions. Where ever currencyId is required or currencyaddress is required, consider naming this explicitly instead of just using the name currency to avoid confusions
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L238 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L307 

This cuts across various areas of the codebase, Only highlighted the above for reference.

3) Missing Zero address check in the following functions

-`WithApproval Modifier in LiqudityPool.sol`
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L97 

Recommendation
Ensure the address is not a zero address

-`Deposit function in LiquidityPool.sol`
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L141 

Recommendation
Ensure the receiver is not a zero address

-`Mint function in LiquidityPool.sol`
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L148

Recommendation
Ensure the receiver is not a zero address

-`Add Pauser in PauseAdmin.sol`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L34 

Recommendation
Ensure address is not a zero address

-`Rely Function in Auth.sol`
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Auth.sol#L14 

Recommendation
Include a check to ensure address is not a zero address

- `addIncomingRouter` function in gateway.sol

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L137

Recommendation
Include a check to ensure router address is not a zero address

- `updateOutgoingRouter` function in gateway.sol

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L147

Recommendation
Include a check to ensure outgoing router address is not a zero address




4)- Inadequate Natspec comment 
Codebase lacks proper documentation on what each function does. Consider documenting all functions and variables with adequate nastpec to help other developers and users reading the codebase to better understand what each function does.

**Overall Assessment**

The audit of the Centrifuge smart contract system aimed to evaluate its security, resilience, adherence to best practices, and code correctness and quality. After a comprehensive analysis, the following overall assessment is provided:

**Security**

The Centrifuge smart contract system exhibits a strong commitment to security. Coding practices, inspired by MarkerDAO's style, emphasize defensive programming with frequent assert statements to handle invalid inputs. The absence of upgradeable proxies and minimal external dependencies further reduces potential attack vectors. The use of the ward pattern for authentication enhances access control, ensuring that only trusted addresses can interact with critical functions. Overall, the security posture of the Centrifuge project is commendable.

**Sound Architecture**

The project's architecture aligns well with established smart contract best practices and general software best practices. The hub-and-spoke model, with RWA pools managed on the Centrifuge Chain, demonstrates a thoughtful approach to managing real-world assets. The absence of upgradeable proxies and the reliance on contract migrations contribute to a robust architecture that minimizes risks associated with upgrades.


**Conculsion**

In conclusion, the Centrifuge project demonstrates a strong commitment to security, with a well-thought-out architecture and a focus on code correctness and quality. Addressing the identified recommendations will further bolster the project's security and usability. Continued collaboration, thorough testing, and documentation improvements are recommended to ensure the project's success.

**Tools used**
All the issues stated above were found during Manual Review

**Time spent on analysis**
15hrs


### Time spent:
15 hours
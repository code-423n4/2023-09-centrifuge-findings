# Centrifuge Smart Contracts Analysis

## Description Overview of Centrifuge Protocol

`Centrifuge` is a `financial platform` that allows securely connecting the `world of cryptocurrencies` with `real-world assets`. Founded in `2017`, it achieved significant milestones, such as, it was the first protocol where `MakerDAO` minted `DAI` against a `real-world asset`. It collaborated with `Aave` to establish a marketplace where individuals can invest in `real-world assets` using cryptocurrencies. Its model functions as a `hub` on its main blockchain, connected to `spokes` on other blockchains, ensuring secure communication between them.

![Centrifuge](https://github.com/catellaTech/Centrifuge/blob/main/Centrifuge.drawio.png?raw=true)

## 1- Centrifuge System Overview

### **Scope**

The scope includes the following `files`:

- **LiquidityPool.sol**:
  this `contract` is an essential part of `Centrifuge's infrastructure` and is used to `create` and `manage` liquidity pools that enable investors to interact with `real-world assets` using `tranche` tokens.

- **InvestmentManager.sol**:
  This is `contract` is a critical component of the `Centrifuge system`, designed to facilitate seamless interactions between `liquidity pools`, the "`Gateway`," and `users`. Its functions enable efficient management of `investment` and `redemption` orders, precise asset and tranche token calculations based on market prices, and secure execution of asset transfers. Serving as the central interface for users, this contract enhances their experience and ensures the integrity of financial transactions within the system, making it indispensable for both `incoming` and `outgoing` investments.

- **PoolManager.sol**: This `contract` plays a crucial role in the administration of `liquidity pools` and `investment options` in a `Centrifuge system`. It enables the `creation` and `management` of `pools`, investment categories, and supported currencies, while also providing functions for secure asset and `tranche token transfers`. This contract is fundamental to the operation and security of the system.

- **Escrow.sol**: This `contract` provides an additional layer of security and control over tokens stored within it. Only authorized addresses can `approve` fund `transfers` from the contract, helping to prevent unauthorized access to the custodied assets.

- **UserEscrow.sol**: The primary `purpose` of the `contract` is to securely hold tokens for specific destinations. It `ensures` that once tokens are transferred into the `escrow`, they can only be transferred out to `pre-chosen destinations`, and this transfer is controlled by designated `wards` or `authorized` parties.

- **Root.sol**: This `contract` is fundamental in the `system` and is a `ward` on all other `deployed contracts`, meaning it has the power to `grant` and `revoke` permissions to other contract addresses.

- **PauseAdmin.sol**: this `contract` is a component that manages the `authority` to pause the "`Root`" contract. It maintains a list of `authorized pausers` who can trigger the `pause` functionality in the "`Root`" contract. This mechanism adds a layer of control and security to the system, allowing for immediate action to halt specific contract functionalities when necessary.

- **DelayedAdmin.sol**: The purpose of the `contract` is to provide a controlled way to manage the `Root` contract's `pausing`, `unpausing`, `scheduling`, and `canceling` of `rely actions`.

- **Tranche.sol**: This is an `ERC20 token` implementation with custom features, including `managing liquidity pools` as trusted forwarders and enforcing transfer restrictions based on the `RestrictionManager`. This provides flexibility and control in token management and transfers.

- **ERC20.sol**: This `contract` provides a comprehensive implementation of the `ERC20 standard` with additional features for managing `authorizations`, `permits`, and `token supply changes`. It is designed to be flexible and secure for various `token-related` operations.

- **RestrictionManager.sol**: This `contract` is designed to work in conjunction with `ERC1404-compatible tokens` to enforce transfer restrictions based on membership status. `Addresses` that are considered members are allowed to receive tokens without restrictions, while `non-member addresses` are subject to restrictions as defined in the `ERC1404 standard`.

- **Gateway.sol**: This `contract` serves as a `bridge` between various `managers` and `routers`. It handles `incoming` and `outgoing` messages, allowing different components to communicate with each other securely.

- **Messages.sol**: This `library` is used for `encoding` and `decoding` messages.

- **Router.sol**: Its `main function` is to `enable communication` between different blockchains within the `Axelar network`.

- **Context.sol**: The `main` utility of this `contract` is to serve as a basic component for other contracts that need easy access to the `sender's address` in the context of a transaction. This is particularly useful for contracts that need to verify the sender's identity or make decisions based on the sender's address.

- **Factory.sol**: This is a set of two smart contracts, "`LiquidityPoolFactory`" and "`TrancheTokenFactory`," along with some `interfaces` and `dependencies`.these two contracts are `factory contracts` responsible for `deploying new liquidity pool` and `tranche token contracts`, configuring their initial settings, and managing `permissions` and `access control`. They rely on external contracts and interfaces for certain functionality and settings.

- **Factory.sol**: This is a set of two smart contracts, "`LiquidityPoolFactory`" and "`TrancheTokenFactory`," along with some `interfaces` and `dependencies`.these two contracts are `factory contracts` responsible for `deploying new liquidity pool` and `tranche token contracts`, configuring their initial settings, and managing `permissions` and `access control`. They rely on external contracts and interfaces for certain functionality and settings.

- **SafeTransferLib.sol**: this `library` provides a way to ensure that `token transfers` and `approvals` are `executed safely` by checking the success of these operations and reverting in case of failure. This helps prevent unexpected issues and enhances the security of token-related operations in smart contracts.

### Privileged Roles
Authentication uses the ``ward`` pattern, in which addresses can be ``relied`` or ``denied`` to get access.

In the `LiquidityPool`, `PoolManager`, `InvestmentManager` contracts administrators can perform critical actions on the contracts, such as

- changing the investment manager and updating the price.
  These functions are executed through the `file function`.

```solidity
2023-09-centrifuge/src/LiquidityPool.sol

    function file(bytes32 what, address data) public auth {
        if (what == "investmentManager") investmentManager = InvestmentManagerLike(data);
        else revert("LiquidityPool/file-unrecognized-param");
        emit File(what, data);
    }
```

```solidity
2023-09-centrifuge/src/InvestmentManager.sol

    function file(bytes32 what, address data) external auth {
        if (what == "gateway") gateway = GatewayLike(data);
        else if (what == "poolManager") poolManager = PoolManagerLike(data);
        else revert("InvestmentManager/file-unrecognized-param");
        emit File(what, data);
    }
```

```solidity
2023-09-centrifuge/src/PoolManager.sol

    function file(bytes32 what, address data) external auth {
        if (what == "gateway") gateway = GatewayLike(data);
        else if (what == "investmentManager") investmentManager = InvestmentManagerLike(data);
        else revert("PoolManager/file-unrecognized-param");
        emit File(what, data);
    }
```
```solidity
2023-09-centrifuge/src/Root.sol

    function file(bytes32 what, uint256 data) external auth {
        if (what == "delay") {
            require(data <= MAX_DELAY, "Root/delay-too-long");
            delay = data;
        } else {
            revert("Root/file-unrecognized-param");
        }
        emit File(what, data);
    }
```

To avoid redundancy in function descriptions, it is worth noting that this `administrative role` is implemented in other `important contracts` within the scope. We reached out to the `Centrifuge team` via Discord to clarify who will be responsible for these roles, and this was their response:

```shell
hieronx — 09/13/23 at 13:15
Likely multisigs on both delayed admin and pause admin
```

Given the level of control that these `roles` possess within the system, `users` must place full trust in the fact that the entities overseeing these roles will always act correctly and in the best interest of the `system` and its `users`.

## 3- Codebase analysis through diagrams.

We have decided to create ``diagrams`` detailing each part of the ``functions`` of the ``smart contracts`` provided by the ``Centrifuge`` protocol and to create a summary of the functionality of each of the ``contracts``. This approach proves to be ``effective`` for ``us`` as it allows us to thoroughly understand all the functionalities of each ``contract`` and visually ``document`` what we have ``comprehended`` through ``diagrams``. Furthermore, we believe that this approach can be useful in enhancing the understanding of the ``contracts`` among ``developers``, ``auditors``, and ``users``.

We've decided to create the most important contracts. Let's take a look at these:

### Contracts:
### 1. LiquidityPool:

![LiquidityPool](https://github.com/catellaTech/Centrifuge/blob/main/CentrifugeLiquidityPool.drawio.png?raw=true)

### 2. InvestmentManager:

![InvestmentManager](https://github.com/catellaTech/Centrifuge/blob/main/CentrifugeInvestmentManager.drawio.png?raw=true)

### 3. PoolManager:

![PoolManager](https://github.com/catellaTech/Centrifuge/blob/main/CentrifugePoolManager.drawio.png?raw=true)

## 4- Architecture Details of Centrifuge system

The ``Centrifuge protocol's architecture`` revolves around ``Pools`` and ``Tranches``, facilitating the issuance of`` real-world asset-backed tokens``. Each Pool, representing a set of assets, offers multiple ``Tranches``, allowing investors to choose varying risk and return profiles within the same asset class. ``Pools`` operate with native currencies, determining Tranche Token decimals and enabling asynchronous deposits and redemptions via an epoch mechanism. Tranche prices, based on ``real-time Net Asset Values``, are calculated on the ``Centrifuge`` system. The protocol supports multiple currencies, ensuring accurate conversions through decimal normalization. Access controls, including ``Root`` and ``PauseAdmin``, safeguard the system, while emergency scenarios are addressed. User flows cover ``Pool`` and ``Tranche`` creation, investment processes, and liquidity management.

### Architecture recommendations
The platform supports a very generic well implementation, and as the team understands, this leads to a broad variety of malicious deployed well risks. It should be a high priority task for the team to find good ways of representing all Well trust assumptions to its users, and expose that through a smart contract UI or an open source front-end.

## 5- Documents
The documentation of the `Centrifuge` project is quite comprehensive and detailed, providing a solid overview of how is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as `diagrams`, to gain a deeper understanding of how different contracts interact and the ``functions`` they implement. With considerable ``enthusiasm``, we have dedicated some days to creating ``diagrams`` for each of the contracts.
We are confident that these ``diagrams`` will bring significant value to the protocol as they can be seamlessly integrated into the existing `documentation`, enriching it and providing a more comprehensive and detailed understanding for `users`, `developers` and `auditors`.

## 6- Systemic & Centralization Risks
Some potential ``systemic risks`` may include:

  - **Price Calculation Failure**: If there are systematic errors in the calculation of ``Tranche prices`` based on the Net Asset Value (NAV) of ``Pool assets``, it could negatively affect all investors and the overall system's integrity.

  - **Malicious Attacks**: Successful attacks or malicious exploits within the protocol could have systemic effects, potentially undermining investor confidence in the C``entrifuge system`` as a whole.

  - **Security Issues**: The discovery of severe security vulnerabilities in the protocol by ``C4 Wardens`` that could be exploited across multiple contracts or tranches could pose a systemic risk.

**Centralization Risk:**

- **Network Control**: If a small group of entities or actors exerts significant control over the majority of ``Pools`` or ``Tranches``, it could lead to excessive ``centralization`` and increase the potential for collusion or manipulation.

- **Tranche Dominance**: If a single entity or group dominates the ownership of most ``Tranche Tokens`` within a specific ``Pool``, it could disproportionately influence the Pool's decisions.

## 7- Monitoring Recommendations

While `audits` help in `identifying` code-level `issues` in the current implementation and potentially the code `deployed` in production, the `Delegate` team is encouraged to consider incorporating monitoring activities in the production environment. Ongoing monitoring of deployed contracts helps identify potential threats and issues affecting production environments. With the goal of providing a complete `security assessment`, the monitoring `recommendations` section raises several actions addressing trust assumptions and out-of-scope components that can benefit from `on-chain monitoring`.

## Time Spent ⏱

A total of `3 days` were dedicated to completing this audit beetween the 5 and 8 hours.


### Time spent:
17 hours
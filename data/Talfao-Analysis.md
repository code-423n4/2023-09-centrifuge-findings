# 1. Description
Centrifuge is the infrastructure that facilitates the decentralised financing of real-world assets natively on-chain, creating a fully transparent market which allows borrowers and lenders to transact without unnecessary intermediaries.

However, this audit contest was focused on investing part of the ecosystem. The process of investing through liquidity pools, in which liquidity is sent to centrifuge pools.
## 1.1 Architecture Overview
### 1.1.1 Liquidity pools (ERC4626 standard)
- Through liquidity pools, the investor could invest in centrifuge pools.
- Each liquidity pool specifies which stablecoin (or other ERC20) token is used for investing. After that, the specific tranche token is emitted as a `share`.
### 1.1.2 Investment manager
- This contract is responsible for the whole investing logic and management of assets and shares which users can claim.
- It communicates with a gateway.
- Shares (tranche tokens) are minted to the escrow contract by the investment manager on behalf of requests coming from a gateway.
- Assets (currencies) are locked for the users in the UserEscrow contract to be withdrawn when this manager gets correct requests from a gateway.
### 1.1.3 Escrow contract
- This contract is used as a temporal depositor of assets which centrifuge pools can take.
- Tranche tokens for the users are minted to this contract, which can be claimed through liquidity pools via the deposit/mint function afterwards and represent the user's shares.
### 1.1.4 UserEscrow contract
- This contract is used for locking assets prepared for withdrawal for the user.
### 1.1.5 Tranche token
- Tranche tokens represent the shares of users.
- One tranch token can be used across many liquidity pools.
### 1.1.6 Restriction manager
- Handle a list of destinations(addresses) which can receive tranche tokens.
### 1.1.7 Pool manager
- This contract is used for creating liquidity pools.
- Adding allowed currencies (ERC20 tokens) in the system.
- Interaction with the restriction manager
- Creating/managing tranche tokens.
- Adding available centrifuge pools to the system.
- Transferring users' tranche tokens to Centrifuge chain or other EVM chains.
### 1.1.8 Gateway
- Used for obtaining incoming messages from routers
- Sending messages to routers.
- Calling functions of different parts of the system based on obtained messages.
### 1.1.9 Routers
- Deliver messages between Gateway and centrifuge pools.
### 1.1.10 Delayed admin
Admin of this contract can pause the protocol and schedule a rely, which makes the supplied user a ward on root contract
### 1.1.11 Paused admin
Pauser of pauseAdmin contract can pause the protocol, and admin can manage pausers.
### 1.1.12 Root contract
The admin of the root contract can make himself or another user an admin of almost every part of the system.

This system works asynchronously, which points out a strong need for the correct access control design.

# 2. Architecture improvements

## 2.1 Missing remove allowed currencies
The pool manager allows the addition of new currency, which will be used in liquidity pools. Consider implementing a mechanism to restrict some currency which is currently used in a system. It can be necessary if some issues with tokens arise in future, which would harm the whole protocol.

## 2.2 The Withdraw function is more reliable than redeeming.
Even though it is clear that liquidity pools are keeping ERC4626 standard, the withdraw function is more reliable than redeem as, from my testing, it prevents possible dust of currency tokens, which can be sustained in the UserEscrow contract. Consider the recommendation for users that it is reliable to withdraw the exact amount of currency.

# 3. Centralisation risks
The protocol realises the risks connected with this asynchronous model of this system. Still pointing them out again.

## 3.1 Compromising of Root
If the root contract is compromised, the attacker can become a ward on any system part via the function `relyContract(target, user)`
## 3.2 Compromising of Investment Manager
If this contract is compromised, an attacker can mint multiple tranche tokens and steal assets from the Escrow contract.
## 3.3 Compromising of Router
If the router is compromised, any message can be delivered to the system, which can cause harm. As the router is communicating with the Gateway, which calls privileged functions of other parts of the system
	
Robust monitoring of every part of the system is needed in case of asynchronous architecture to mitigate any possible malicious behaviour.
Also, Multisig wallets should be used for peripheral EOA accounts such as delayed admin or paused admin.

# 4. Time spent
I have spent four days with this audit.
1. On the first day, I read all docs provided for this contest.
2. On the second day, I checked the code and tests.
3. On the third day, I tried different attack scenarios which came to my mind but without success. Reread a code and try to link functionality together.
4. On the fourth day, I summarised the audit via this analysis report. 


### Time spent:
20 hours
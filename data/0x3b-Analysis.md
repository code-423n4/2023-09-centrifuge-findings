# Summary
The audit unveiled a well-structured and intriguing codebase, primarily centered around economic aspects. While the code demonstrated promise, there were minor concerns regarding code quality and documentation. Additionally, the architecture exhibited potential, particularly in the proposal to introduce a flywheel mechanism to incentivize users. The audit also raised awareness about centralization risk, emphasizing the need to strike a balance between centralization and decentralization for optimal system resilience.

| Severity | Occurrences |
|----------|-------------|
| High     | 1           |
| Medium   | 7           |
| Low      | 1           |
| Gas      | 0           |
| Analysis | 8 hours     |


# Time allocations

|            |            |
|------------|------------|
| Start date | 08.09.2023 |
| End date   | 14.09.2023 |
| **Total**  | **6 days** |

| Members       | Positions            | Time spent |
|---------------|----------------------|------------|
| **0x3b**      | full time researcher | ~50H       |

# Findings
The codebase was "a blast," and the time spent was just perfect. I had enough time to delve into every aspect of the scope, even exploring some out-of-scope codebases to gather reference points. The economics were truly fascinating, and I examined them from every perspective. The bridge was intriguing, though unfortunately, the other part of the code was out of scope. In fact, approximately 80% of my time was dedicated to attempts to manipulate the rewards, deposits, and withdrawals. I encountered several scenarios where any user could potentially cause damage, extract value, and achieve a better profit by manipulating the price to their advantage.

# Codebase quality
The code quality was excellent, however, it contained some unnecessary functions that only increased gas consumption during deployments. Additionally, it lacked proper testing. The bridge part was highly sophisticated, however again, due to the numerous missing pieces of information, I suspect that some of the findings may be invalid. In general, there might be many false positives or unexpected behaviors. For future reference, it would be much better to have more natural language specifications or more documentation on the matter.


# Mechanism review

###### LiquidityPool.sol
- One of the main contracts.
- The point from which users interact with the system.
- ERC4626 compatible.
- Lacks an oracle; however, it uses the bridge prices to convey value.
- Contains zero logic, used as an interface.
- It wraps [**InvestmentManager**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol) for interface-like usage.

###### InvestmentManager.sol
- Main contract where the logic is located.
- Can only be called by wards, not users.
- Receives function calls from [**LiquidityPool**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol), which is its interface-like contract.
- View functions can be accessed by users.
- Has maximum deposit/withdrawal values to convey price.
- Communicates with the gateway.

###### Escrow.sol
- Contract where user funds are held.
- Receives funds both from [**InvestmentManager**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol) when funds must be bridged to Centrifuge and from Centrifuge when they must be sent back.
- Has only one function, `approve`, which approves different contracts to spend its tokens.
- Extremely secure on its own.

###### PoolManager.sol
- Manager of pools.
- Creates pools, allows different currencies, and adds tranches for them.
- Stores information about the pools.

###### UserEscrow.sol
- Closely resembles [**Escrow**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Escrow.sol).
- Has only two functions, [transferIn](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L29-L34) and [transferOut](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L36-L50), for handling tokens.

###### Root.sol
- Owner-like contract.
- Main ward contract.
- Provides access-modifying functions that can grant or remove roles for different users.
- Includes a mechanism to pause the system.

###### PauseAdmin.sol
- Contract to assign or remove different contracts from the pauser role.
- Able to pause the system.

###### DelayedAdmin.sol
- Access point for EOA wards, also called "delayed admins".
- Can pause the system.
- Can schedule someone to become a "delayed admin".
- A security measure requires some time to pass before making someone a "delayed admin".

###### Tranche.sol
- ERC20 shares contract.
- Also implements ERC1404.
- Includes checks to verify if a user is a member (KYS account).

###### ERC20.sol
- ERC20 contract used for [**Tranche**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol).
- Implements the AUTH modifiers.
- Has a domain separator with [_calculateDomainSeparator](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L67-L77).
- Implements other basic ERC20 functions.
- Includes a permit function allowing users to grant allowances without making transactions.

###### RestrictionManager.sol
- A contract that manages the member state.
- Members can be added by the ward, which in this case is the pool issuer.

###### Gateway.sol
- A contract for handling calls to the gateway.
- Utilizes [**Messages**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol) to encode/decode the parameters.
- A straightforward contract with repetitive functions.
- Contains a [handle](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol#L285-L366) function for receiving calls from the other chain.

###### Messages.sol
- Used by [**Gateway**](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol).
- Features interesting call management with the [call](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L9-L61) enum.
- A simple contract with repetitive functions.
- Includes a basic packing mechanism.

###### Router.sol
- An Axelar gateway contract.
- A small and straightforward contract.
- Does not pay fees to Axelar; they are managed by the backend Centrefuge bot.
- Includes a modifier to check if the call is made to Centrefuge.

###### Auth.sol
- A basic contract for ward management.
- Implemented by every other contract.
- Features a simple allow/deny mechanism.

###### Context.sol
- An OpenZeppelin contract.
- Contains only one function - to retrieve the `msg.sender`.

###### Factory.sol
- Consists of two factory contracts.
- One for creating tranches and another for creating pools.
- Utilizes salt, which is a plus.
- After creation, removes itself from the wards.

###### SafeTransferLib.sol
- An OpenZeppelin-modified safe transfer helper function.
- Used to ensure that the ERC20 transfer has succeeded.

# Architecture recommendations
The contracts provided to us were sufficient to achieve the system's intended goals. They included managers, wrappers, and bridges. They were relatively simple, and the code was straightforward, making the audit a pleasant experience. 

One potential enhancement I would recommend is implementing a flywheel mechanism to further incentivize and encourage users to stake on Centrefuge. 

Another alternative worth considering is incorporating an NFT concept. Users could lock their stakes and receive a minted NFT, which they could choose to sell or hold. This approach could introduce an intriguing dimension to the system and offer users additional benefits for their participation in Centrifuge. It's important to note that I am not a consultant on stocks and bonds, and I cannot say whether these suggestions would comply with the law.

# Centralization risk
The system's centralization risk is significant due to the absence of any form of governance. It does have admins, including both EOAs and other contracts. However, this centralization may not be a major concern, provided that these admins are genuinely invested in the project's overall growth. When harnessed effectively, centralization can provide the system with a competitive edge over its competitors. It facilitates rapid decision-making, enabling Centrefuge to adapt swiftly and stay ahead of industry trends. In contrast, decentralized DAO systems often require proposals, votes, and lengthy approval and implementation processes.

Nevertheless, it's crucial to acknowledge that centralization also carries potential downsides, such as an increased vulnerability to single points of failure. Striking a balance between centralization and decentralization is important to ensure the system's robustness and sustainability. One recommended approach is to implement a multisig system involving a small group of developers/owners who can collectively make decisions. This approach ensures that the protocol remains secure even if a private key is leaked or an account is compromised, while still maintaining agility.

However, it's worth recognizing the developers' efforts in mitigating the impact of malicious admins. From the code, it's apparent that the most they can do before being removed is to pause the system, which is not a significant issue. On the other hand, the EOA method also enables quick responses to emergencies since, in the event of one, an EOA can promptly pause the entire system.

# Systemic risks
The current state of this protocol leaves it susceptible to various types of economic attacks. Certain functions within the code can be manipulated by users to gain advantages over others or even illicitly acquire tokens from the protocol. However, after a comprehensive refinement following this audit, the risks associated with these vulnerabilities can be significantly reduced. By addressing these weaknesses and implementing robust security measures, the protocol can become more resilient against potential economic attacks.

Nonetheless, it is crucial to exercise caution when adding new features to the code, especially those involving economic incentives. While they can enhance the protocol's performance and attract users, they also tend to be the most vulnerable points, susceptible to exploitation if not designed and implemented with the utmost care.

### Time spent:
-50 hours
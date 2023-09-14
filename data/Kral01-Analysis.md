## 1. Analysis of Codebase

The codebase for Centrifuge's Liquidity Pools is well-structured and effectively integrates cross-chain communication. This structure aligns with Centrifuge's mission to bridge businesses or "Asset Originators" and investors through Decentralized Finance (DeFi). Here is a step-by-step process flow of the protocol from a user's perspective:

1. Users initiate a deposit into the protocol via the 'LiquidityPool' contract by sending deposit requests.
2. The 'InvestmentManager' contract handles and validates these requests.
3. The deposit amount is held in escrow by the 'UserEscrow' contract.
4. The deposit requests are encrypted and sent to the Centrifuge chain by the Gateway.sol and messages.sol contracts.
5. The Centrifuge chain processes these requests, and the 'InvestmentManager' contract handles incoming messages.
6. All operations such as `deposit`, `redeem`, `mint`, `withdraw` etc., follow this execution order.
7. The protocol also provides a feature to decrease your deposit or redeem request while the request is in process.

Access controls are well-implemented using 'root.sol', which serves as the central access control entity for all contracts through a ward feature. The tests are comprehensive and utilize fuzzing, providing a robust coverage. The gateway routers adhere to best practices, and the ERC4626 implementations are mostly compliant with recommended guidelines.

## 2.Architecture recommendations

1. Consider changing the visibility of functions that are not called by other contracts to internal, as most functions in all contracts are currently public.
2. It might be beneficial to restrict the creation of new pools to members only, even though there aren't immediate threats since tranches and currencies are validated.
3. While the ERC-4626 implementation is mostly correct, there are some missing elements highlighted in the report, and it would be beneficial to address them.
4. Consider conducting a gas audit. Improvements can be made to decrease the gas usage of the codebase, providing a better user experience.


## 3.Centralization risks

The protocol has a few centralization risks that should be addressed:

1. Admins currently have the ability to pause the Liquidity Pools at any time.
2. The root.sol contract is a central point of control that manages all other contracts and access controls. This presents a risk as any member can be removed at any time. Given that user funds can be locked in the contract if they are removed, this issue should not be overlooked.

## 4.Time Spent

The audit took a total of 4 days:

1st Day: Understanding the protocol's documentation, process flow, and end user flow, and taking notes and drawing diagrams. 
2nd Day: Linking the documentation's logic to the codebase by following the user's entry point, scanning for vulnerabilities by executing the user flow, taking notes of potential vulnerabilities, and reading reports for similar vault-based protocols. 
3rd Day: Focusing on different types of attack vectors and documenting found vulnerabilities. 
4th Day: Summarizing the audit by completing the QA report and analysis.




### Time spent:
48 hours
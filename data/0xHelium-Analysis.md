# Analysis report for Centrifuge

## Codebase quality
The overall quality of the codebase for PoolTogether can be classified as "Good".

Strengths

Natspec was really helpful and detailed.
It tries to achieve complete decentraliztion. It’s remarkable they can do this without any governance.
Documentation was 100% on point for the codes in scope.
Weaknesses.

Majority of the tests were using mock addresses instead of actual contracts. This is not recommended.

## Mechanism review
Centrifuge works based on a hub-and-spoke model. RWA pools are managed by borrowers on Centrifuge Chain, an application-specific blockchain built purposely for managing real world assets. Liquidity Pools are deployed on any other L1 or L2 where there is demand for RWA, and each Liquidity Pool deployment communicates directly with Centrifuge Chain using messaging layers.

## About Centrifuge:

Founded in 2017
An institutional platform for onchain credit
Pioneered MakerDAO's DAI minting against real-world assets
Launched the first onchain securitization
Introduced the RWA Market with Aave
Utilizes a multi-chain strategy for native RWA yields
Contract Overview:

## How Centrifuge operate
Centrifuge operates on a hub-and-spoke model
Real-world asset pools managed on Centrifuge Chain
Liquidity Pools deployed on various L1 or L2 networks
Tranches enable investors to access different risk exposure and yield
High-Level Contracts:

## Contracts overview
Investment Manager: Core business logic for pool management
Pool Manager: Handles currency bookkeeping and transfers
Gateway: Encodes and decodes messages for communication
Routers: Handle message communication with Centrifuge Chain

## Multiple Currency Support:

Liquidity Pools support deposits in various currencies
ERC20 methods passed through to the underlying share implementation
Price calculations account for differences in decimals

## Access Setup:
Root contract acts as a ward on all other contracts
PauseAdmin can pause the protocol
DelayedAdmin can become a ward through Root.relyContract
Emergency scenarios are addressed through careful governance

## User Flows:
Pools and tranches are created and deployed for real-world assets
Users can invest in different tranches for risk exposure and yield
Liquidity management involves currency swaps and escrow contracts

## Systemic risks:
Like any smart contract-based system, PoolTogether is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol.
The communication between various chains could lead at a DoS if any chain go ghost or disappear for any reason

## Architecture recommendations:
I recommend rewriting the tests in the codebase for this audit to use the actual contracts instead of mock addresses. This will offer greater confidence during system deployment.
The investor in tests should be an eoa and not a smart contract.
Unfortunately i was not able to rewrite the tests due to time constraint.

##Approach for Centrifuge Audit
Objective:
The primary focus of this audit is to thoroughly assess the Centrifuge protocol, with specific attention to the core contracts, security mechanisms, and potential vulnerabilities.

##Audit Timeline:
###Day 1 - System Familiarization:

Gain a comprehensive understanding of the Centrifuge protocol, its architecture, and its overall functionality.
Review documentation and whitepapers to establish a baseline knowledge.
Begin preliminary codebase exploration to identify key contract components.

##Day 2 - Contract Analysis:
Deep dive into the core contracts and smart contract interactions within the Centrifuge protocol.
Identify and document critical contract functions, state variables, and external dependencies.
Focus on Investment Manager, Pool Manager, Gateway, and Routers contracts, their roles, and security measures.
Assess contract upgradeability and authentication mechanisms.

##Day 3 - Security Assessment:
Identify potential attack vectors, edge cases, and vulnerabilities in the Centrifuge protocol.
Thoroughly analyze the liquidity pool mechanisms, deposit, and redemption processes.
Investigate the handling of multiple currencies and associated conversions.
Review the access setup and governance mechanisms for potential weaknesses.
Create Proof of Concepts (PoCs) to demonstrate any identified vulnerabilities.
Assess the protocol's resistance to common attack scenarios.

##Day 4 - Report and Recommendations:
Compile findings and vulnerabilities discovered during the audit.
Prioritize and categorize vulnerabilities based on their severity.
Provide detailed recommendations for mitigating identified risks.
Summarize the audit results, highlighting key areas of concern.







### Time spent:
20 hours
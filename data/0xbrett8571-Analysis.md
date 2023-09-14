**Introduction**

Centrifuge is a decentralized protocol that enables institutional lenders to issue and manage real-world assets (RWAs) on-chain. RWAs are assets that have value in the real world, but are not easily traded on traditional financial markets. Centrifuge uses a hub-and-spoke model, with RWA pools managed on Centrifuge Chain and Liquidity Pools deployed on other blockchains.

**High level contract overview**
![Screenshot 2023-09-14 145247](https://github.com/code-423n4/2023-09-centrifuge/assets/125544245/6f4c3228-3b7e-4c84-b532-a9384b98e43a)

**Codebase Review**

The Centrifuge codebase is well-written and well-organized. It uses a modular design with clear separation of concerns. The code is well-documented and easy to understand.

The full relationships of `wards` can be seen below.

![Screenshot 2023-09-14 145334](https://github.com/code-423n4/2023-09-centrifuge/assets/125544245/f6461efb-b1aa-4944-bd11-b8402d70b6ab)

**User flows**

How pools and tranches are created and deployed

![Screenshot 2023-09-14 145413](https://github.com/code-423n4/2023-09-centrifuge/assets/125544245/a35068e1-89b6-4b93-9ce7-930793b61ef7)

**How users can invest**

![Screenshot 2023-09-14 145437](https://github.com/code-423n4/2023-09-centrifuge/assets/125544245/d4c3126c-8677-4284-a47f-e04b4d092f82)


**Centralization Risks**

There are a few centralization risks associated with Centrifuge. First, the PauseAdmin has the ability to pause the protocol at any time. Second, the Root contract is a ward on all other contracts, which gives it a lot of power. Third, the Gateway contract is controlled by a small number of routers.

**Mechanism Review**

The Centrifuge mechanism is complex, but it is well-designed and secure. The use of an epoch mechanism and messaging layers helps to mitigate centralization risks.

**Systemic Risks**

There are a few potential systemic risks associated with Centrifuge. First, if a large number of borrowers default on their loans, it could lead to a liquidity crisis. Second, if a large number of investors withdraw their funds from Liquidity Pools, it could also lead to a liquidity crisis.

**An example flow for how this works is visualized below**

<img width="1386" alt="liquidity_flow1" src="https://github.com/code-423n4/2023-09-centrifuge/assets/125544245/d75bb74c-3637-45d9-8ed2-fd279218d6af">

<img width="1274" alt="liquidity_flow2" src="https://github.com/code-423n4/2023-09-centrifuge/assets/125544245/40481632-257e-4844-8dbc-20a7b01ceb35">

**Comments for the Judge**

Overall, the Centrifuge codebase is well-written and well-organized. There are a few centralization risks associated with the protocol, but the mechanism is well-designed and secure. The potential systemic risks are mitigated by the use of an epoch mechanism and messaging layers.

**Recommendations**

* The PauseAdmin should be replaced with a more decentralized mechanism for pausing the protocol.
* The Root contract should be split into multiple contracts with different roles and responsibilities.
* The Gateway contract should be controlled by a larger number of routers.
* The protocol should be audited by a reputable security firm.

**Architecture Recommendations**

* The PauseAdmin should be replaced with a multi-sig wallet controlled by a group of trusted entities.
* The Root contract should be split into the following contracts:
    * InvestmentManager: Handles pool creation, tranche deployment, and managing investments.
    * PoolManager: Handles currency bookkeeping and transferring tranche tokens and currencies.
    * Gateway: Encodes and decodes messages using Messages and handles routing to/from Centrifuge.
    * Routers: Handle communication of messages to and from Centrifuge Chain.
* The Gateway contract should be controlled by a DAO or consortium of institutions.

**Conclusion**

Centrifuge is a promising project with the potential to revolutionize the way institutional lenders manage RWAs. However, there are a few centralization risks and potential systemic risks associated with the protocol. These risks can be mitigated by implementing the recommendations outlined above.

**Additional Comments**

* The Centrifuge team is actively working to address the centralization risks and potential systemic risks associated with the protocol.
* The Centrifuge team is also working on new features, such as cross-chain liquidity and support for more types of RWAs.

I believe that Centrifuge has the potential to become a major player in the institutional credit on-chain ecosystem. The team is committed to building a secure and reliable platform for lenders and investors.

### Time spent:
37 hours
0. QA/LOW: permit() function enables possibility of griefing attack on permit calls via frontrunning attack vector.

https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L10
https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L223
https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L240

Description
Within the context of the ERC-4626 compatible contract "LiquidityPool," designed to facilitate stablecoin investment in tranches of pools, there is a specific concern related to the "permit" function. This function allows for potential front-running, potentially disrupting the workflow of third-party smart contracts involved in the investment process.

In the "LiquidityPool" contract, the "permit" function is part of the ERC20PermitLike interface, enabling changes to a user's allowance through a signature check, using ecrecover for validation purposes. While this functionality is correctly implemented within the contract, it presents a security issue that developers and users of contracts interacting with the pool tokens should be aware of.

Security Considerations
One security consideration pertains to the potential front-running of "permit" transactions. Although the signer of a "Permit" may have a specific intended party to submit their transaction, there exists a risk that another party can preemptively call the "permit" function before the intended transaction, affecting the outcome. While the ultimate result *could* remain the same for the "Permit" signer, this front-running behavior *could* disrupt the intended execution flow.

Exploit Scenario
To illustrate this potential risk in the context of the "LiquidityPool" contract for stablecoin investment tranches, let's consider an example: Alice develops a smart contract that leverages the "permit" function to facilitate a transferFrom of USDC without requiring users to execute an approve transaction beforehand. Eve, who is actively monitoring the blockchain, detects this "permit" call. She carefully observes the signature and proceeds to front-run Alice's call, effectively causing a revert in Alice's contract and preventing its expected execution.

Recommendation
In the short term, it is essential to thoroughly document the possibility of "permit"-related front-running attacks to alert users and developers who interact with the pool's tokens. By providing this information, users can anticipate the potential risks and develop alternative workflows or mitigation strategies in case they become targets of such attacks.

In the long term, ongoing vigilance is crucial. Monitoring the blockchain for signs of front-running attempts and promptly responding to potential threats is essential for maintaining the security and integrity of the ERC-4626 compatible contract "LiquidityPool" and the stability of the tranches of pools it facilitates.

=======
submitted above.
=======

1. QA: LiquidityPool::constructor() - Input validation: How will the protocol avoid to pass existing/invalid `poolId_` or `trancheId_` parameters in constructor() during LiquidityPool deployment?

https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L86

2. QA: LiquidityPool::constructor() - Input validation: Seems there are no checks to ensure that same `asset` & LP pool combo doesnt already exist for same tranche(RWA) centrifuge pool, and also no address(0) check nor any check for decimals over 18.

https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L88

3. QA: LiquidityPool::constructor() - Input validation: Consider adding a check to ensure asset & share token addresses are not the same. Also, no address(0) check.

https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L89

4. QA: LiquidityPool::constructor() - Input validation: consider adding address(0) check for parameter `investmentManager_`.

https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L90

5. QA: LiquidityPool::withApproval() - Unclear dev comment, suggest to improve for clarity & to eliminate any confusion.

https://github.com/code-423n4/2023-09-centrifuge/blob/882f620bd3e92015f30af74266b70a1827db97fb/src/LiquidityPool.sol#L96

Current dev comment:
"Either msg.sender is the owner or a ward on the contract"

Recommendation:
"Either msg.sender is the owner of the contract or msg.sender is a ward on the contract"


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


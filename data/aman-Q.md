# [QA:1] Wrong Comment 
Severity: Informational

The comments in the smart contract indicate that the modifier should validate whether the msg.sender is either the owner or part of the wards. However, upon review, it appears that there is no implemented code within the modifier to manage the wards' flow. This discrepancy needs to be addressed to ensure the contract functions as intended.

Recommendation: We have identified what appears to be an error in the comments section of the smart contract. Consequently, it is necessary to remove the erroneous segment to maintain clarity and accuracy.


# [QA:2] lastPriceUpdate not used in contracts
Severity: Low

In the LiquidityPool.sol file, the state variable 'lastPriceUpdate' is initialized at line https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L75C20-L75C35
. However, it appears that this variable is not utilized within any smart contracts. It is plausible that its application may be off-chain.

Recommendation: Upon reviewing the existing code, it was observed that an event for price update is already in place, which is triggered whenever a price update occurs. However, it is recommended to incorporate 'lastPriceUpdate' into the 'UpdatePrice' event for enhanced tracking and transparency. The revised event should appear as 'UpdatePrice(lastPriceUpdate, price)'.



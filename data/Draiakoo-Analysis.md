1-In Root.sol, the escrow address is defined in the constructor but is never used. https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L19
Recommendation: either remove it or leave it as it is if you want to implement further logic involving this address

2-The logic of the modifier inside LiquidityPool.sol withApproval does not match the comment that has above. The comment says that checks if the msg.sender is either the owner or a ward. However, only checks if it is the owner. https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol#L96-L100
Recommendation: remove the part of the comment that says "or a ward on the contract"

3-Functions _toPriceDecimals() and _fromPriceDecimals can be changed from view to pure, because thes does not read from any stored value. https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L674-L693

### Time spent:
20 hours
# QA reports

1. [QA reports](#qa-reports)
   1. [L-1 `LiquidityPool.permit` does not work with DAI token.](#l-1-liquiditypoolpermit-does-not-work-with-dai-token)
   2. [L-2 `UserEscrow.transferOut()` issue with require check `ERC20.allowance == type(uint256).max`](#l-2-userescrowtransferout-issue-with-require-check-erc20allowance--typeuint256max)

## L-1 `LiquidityPool.permit` does not work with DAI token

Dai is the main investment token of Centrifuge. It is weird to see new crosschain investment not supporting the main staple coin.
DAI permit function predate `EIP-2612` so it need custom permit function to work with. ([Reference Code](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L10-L11),[Detail 1](https://github.com/d-xo/weird-erc20/tree/main#unusual-permit-function), [Detail 2](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2206))

## L-2 `UserEscrow.transferOut()` issue with require check `ERC20.allowance == type(uint256).max`

Some token like Uniswap token can only return `uint96.max` as allowance.
This limit user from redeem/withdraw to another address beside original owner.
[Reference Code](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L43)

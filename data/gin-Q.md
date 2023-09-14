[L1]Function name is safeTransfer/safeTransferFrom but low-level call transfer/transferFrom
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/SafeTransferLib.sol#L15-L19
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/SafeTransferLib.sol#L26-L29

In both safeTransferFrom and safeTransfer function, token calls IERC20.transfer and IERC20.transferFrom, does not uses SafeERC20 library as the function name implies.

Mitigation:
Use safeERC20 library instead

[L2]increaseAllowance/decreaseAllowance is deprecated
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L139
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L148

Check out openzeppelin-contracts PR:
https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4583

safeIncreaseAllowance/safeDecreaseAllowance is still available in SafeERC20, so consider change increaseAllowance/decreaseAllowance to safeIncreaseAllowance/safeDecreaseAllowance
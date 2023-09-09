## [L-01] Unsafe ERC20 Operation(s)
Description
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.
```txt
2023-09-centrifuge/src/InvestmentManager.sol::167 => lPool.transferFrom(user, address(escrow), _trancheTokenAmount);
2023-09-centrifuge/src/InvestmentManager.sol::313 => LiquidityPoolLike(liquidityPool).transferFrom(address(escrow), user, trancheTokenPayout),
2023-09-centrifuge/src/InvestmentManager.sol::475 => lPool.transferFrom(address(escrow), user, trancheTokenAmount),
2023-09-centrifuge/src/PoolManager.sol::133 => gateway.transfer(currency, msg.sender, recipient, amount);
2023-09-centrifuge/src/PoolManager.sol::249 => EscrowLike(escrow).approve(currencyAddress, investmentManager.userEscrow(), type(uint256).max);
2023-09-centrifuge/src/PoolManager.sol::252 => EscrowLike(escrow).approve(currencyAddress, address(investmentManager), type(uint256).max);
2023-09-centrifuge/src/PoolManager.sol::261 => EscrowLike(escrow).approve(currencyAddress, address(this), amount);
2023-09-centrifuge/src/PoolManager.sol::328 => EscrowLike(escrow).approve(liquidityPool, address(investmentManager), type(uint256).max); // Approve investment manager on tranche token for coordinating transfers
2023-09-centrifuge/src/PoolManager.sol::329 => EscrowLike(escrow).approve(liquidityPool, liquidityPool, type(uint256).max); // Approve liquidityPool on tranche token to be able to burn
2023-09-centrifuge/src/token/Tranche.sol::60 => return super.transfer(to, value);
2023-09-centrifuge/src/token/Tranche.sol::69 => return super.transferFrom(from, to, value);
```
## [L-02] Do not use Deprecated Library Functions
Description
The usage of deprecated library functions should be discouraged.
This issue is mostly related to OpenZeppelin libraries.
```txt
2023-09-centrifuge/src/Escrow.sol::24 => SafeTransferLib.safeApprove(token, spender, value);
2023-09-centrifuge/src/util/SafeTransferLib.sol::36 => function safeApprove(address token, address to, uint256 value) internal {
```
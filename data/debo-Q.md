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
## [L-03] A control flow decision is made based on The block.timestamp environment variable
Description
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
Vulnerable Code 1
// https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L46
```sol
require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");
```
Exploit Code 1
```sol
function testFailMember() public view {
   RestrictionManager.member(0x0000000000000000000000000000001000000000);
}
```
Vulnerable Code 2
// https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L50
```sol
        if (members[user] >= block.timestamp) {
```
Exploit Code 2
```sol
function testFailHasMember() public view {
   RestrictionManager.hasMember(0x0000000000000000000000000000000400008004);
}
```
Vulnerable Code 3
// https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L58
```sol
   require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");
```
Exploit Code 3
```sol
function testFailUpdateMember() public view {
   RestrictionManager.updateMember(0x0000000000000000000000000000000000000000, 0);
}
```
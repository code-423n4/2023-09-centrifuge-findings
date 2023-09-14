
1. The `member` view function does not return anything:

```solidity
function member(address user) public view {
        require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");
    }
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L45-L47

2. Missing address(0) check in contract constuctions and functions like `addLiquidityPool`, `removeLiquidityPool`, etc

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L48-L56

3. It is very recommended to use ECDSA lib to prevent several signature related vulnerabilities

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L206-L207

4. The `name`, `symbol` remains unintialized as it is never assigned a value.

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L19-L20

5. Self-transfers can be prevented by adding check:

`to` != `msg.sender`

and

`to` != `from`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L90-L129

6. `_delay` is not compared with `MAX_DELAY` and could be set to a value even greater.

The following check is missing:

`require(_delay <= MAX_DELAY, "_delay is greater than maxDelay")`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L36-L37

7. The `pause()` should execute only if `paused == false`, to prevent unnecessary emiting event, and vice-versa for `unpause()`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L54-L62

8. The `addIncomingRouter()` should execute only if `router == false`, to prevent unnecessary emiting event, and vice-versa for `removeIncomingRouter()`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L137-L145

9. The `addPauser()` should execute only if `pausers[user] = 0`, to prevent unnecessary emiting event, and vice-versa for `removePauser()`

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L33-L42
## [info] `liquidityPool` maybe can burn itself, does not need to approve itself
   `liquidityPool` maybe can burn itself, does not need to approve itself, I suggest we can delete this code
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L329

## [low] `UserEscrow.transferOut()` if using the approve, it must approve all, this is unreasonable
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L36

`UserEscrow.transferOut()` if using the approve, it must approve all, this is unreasonable. Maybe the user just want to approve  a part of fund.I suggest change it to :
```solidity
require(
            /// @dev transferOut can only be initiated by the destination address or an authorized admin.
            ///      The check is just an additional protection to secure destination funds in case of compromized auth.
            ///      Since userEscrow is not able to decrease the allowance for the receiver,
            ///      a transfer is only possible in case receiver has received the full allowance from destination address.
            receiver == destination || (ERC20Like(token).allowance(destination, receiver) >= amount),
            "UserEscrow/receiver-has-no-allowance"
        );
```

## [info] No one calls the `removeLiquidityPool()` function
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol#L53

I am not sure the intention of coder. But if the function is useless ,I suggest we delete it or write a remark
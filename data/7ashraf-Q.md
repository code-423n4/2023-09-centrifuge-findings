

# L-01 Check if the router exists before removing 
A check should be made before removing a router removal is done
## Instances 
Gateway.sol #142 
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L142C1-L145C6

`
    function removeIncomingRouter(address router) public auth {
    incomingRouters[router] = false;
        emit RemoveIncomingRouter(router);
    }
`
## Impact 
A caller may enter an invalid router address and it will still be removed ie nothing will happen, but the event will still be emitted that the router has been removed, emitting a false event in which the router did not exist in the first place, also a caller may enter a typo on the removed router causing it to emit that it has been removed and he will think that it has been removed correctly.


# L-02 ward can remove all wards including himself
## Impact
A certain ward can remove all wards including himself from the ward contract which will cause a dos if the ward account is stolen or under any other reasons.
mitigation: should not allow wards to remove other wards except for himself if he chooses to
add another higher role of wards which can remove all wards if needed

# L-03 Some restrictions should be applied to the token name and symbol
## Instance
Factory.sol #98 
## Impact
 No restrictions are done on the token name and symbol allowing the user to use nonsense names and symbols, possibly the empty string which may cause some problems if used
mitigation: require that the symbol and token name follow a certain amount of rules for safety



# L-04 confusing naming of the function 
## Instance
InvestmentManager.sol  _decreaseDepositLimits, _decreaseRedemptionLimits
## Impact
The function behavior may be confusing out of the naming alone, a developer or a user may be confused if the function will make him be able to deposit more or deposit less, as increasing the limit makes him deposit less while increasing the limit value itself will make him deposit more.








# L-05 Separate the logic for a clearer more understandable code
## Instances
 Gateway.sol handle() function #285
 https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L285
the function does a lot of functionalities making it harder to read or understand
## Mitigation 
the function should separate the logic of getting the message type into another function getMessageType() and the message type should be returned, this way the function enables separation of concerns making the code more clear and readable.



# L-06 Use SafeTransferLib for consistency, instead of transferFrom
## Instance
InvestmentManager.sol #475
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L475


    function _deposit(uint128 trancheTokenAmount, uint128 currencyAmount, address liquidityPool, address user)
        internal
    {
        LiquidityPoolLike lPool = LiquidityPoolLike(liquidityPool);

        _decreaseDepositLimits(user, liquidityPool, currencyAmount, trancheTokenAmount); // decrease the possible deposit limits
        require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");
        require(
            lPool.transferFrom(address(escrow), user, trancheTokenAmount),
            "InvestmentManager/trancheTokens-transfer-failed"
        );

        emit DepositProcessed(liquidityPool, user, currencyAmount);
    }




# L-07 overflow may happen causing the calling address to be address(0x0)
## Instances GateWay.sol #285
# L-08 open To-Do
## Instances 
Messages.sol #149
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L149C9-L151C83

`// TODO(nuno): Now, we encode `tokenName` as a 128-bytearray by first encoding `tokenName`
        // to bytes32 and then we encode three empty bytes32's, which sum up to a total of 128 bytes.
        // Add support to actually encode `tokenName` fully as a 128 bytes string.`
        
Messages.sol #739
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L739C8-L741C83
` // TODO(nuno): Now, we encode `tokenName` as a 128-bytearray by first encoding `tokenName`
        // to bytes32 and then we encode three empty bytes32's, which sum up to a total of 128 bytes.
        // Add support to actually encode `tokenName` fully as a 128 bytes string.`
# L-09 follow the same way of creation for consistency and clash avoidance 
## Instance 
Factory.sol #36
the create tranche token is created perfectly using salt to avoid clashes, the liquidity pool should be created the same way for more consistent code functionality and the same reason for clash avoidance


# L-10 factory.sol needs to be restructured
the factory.sol has two contracts for deploying liquidity pool and tranche tokens, which do not need complexity and could be converted into methods, for more simple readable code.


# L-11 Add more comments to explain the assembly code 
## Instances:
RestrictionManager.sol #103
Tranche.sol #107
LiquididtyPool.sol #340
Messages.sol # #882












# L-12 user can transfer funds back and forth to himself

## Instance 
UserEscrow.sol #29 #36
## Proof of concept:
There is no checker that the sender is the receiver
## Impact 
This is not an issue and will cause no harm to the system, but for system versatility and sanity users shouldn't be expected to transfer funds back and forth to themselves

# L-13 Auth.sol contract does not have a ward for its self


# L-14 deploying LiquidityPool does not add msg.sender as a ward 
deploying of a new LoquidityPool using the deployLiquidityPool function at the PoolManager.sol does not add the msg.sender as a ward on the liquidity pool
## Instances
PoolManager.sol #307






# L-15 utilize the use of error() across the whole project as it is not used once 


## Delay could be set to a higher value than MAX_DELAY 

## Description

The variable `delay` is initialized in the constructor, however no checks are made to ensure that the delay period is indeed smaller than the `MAX_DELAY`.

## Links

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34-L35

## Fix

Consider adding a check making sure the `delay` value doesn't exceed `MAX_DELAY`
`require(_delay < MAX_DELAY, "delay value too big")`



## Remove commented code 

## Description
For better clarity remove useless comments.

## Links

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L148-L149

## Fix

remove the useless comments.

## Possible replay attack

## Description
The ERC20 contract allows a user to use the `permit` function with as long as it is within a valid deadline. However, the deadline is not enough to prevent an actor from increasing its allowance, fire the `transferFrom` function, increase the allowance and perform the transferFrom() transaction again and again until the deadline is reached.  
## Links
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L106-L119
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L196-L214
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L216-L243
## Fix
I would recommend making sure that every signature is only playable once. 
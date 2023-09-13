## Delay could be set to a higher value than MAX_DELAY 

## Description

The variable `delay` is initialized in the constructor, however no checks are made to ensure that the delay period is indeed smaller than the `MAX_DELAY`.

## Links

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34-L35

## Fix

Consider adding a check making sure the `delay` value doesn't exceed `MAX_DELAY`
`require(_delay < MAX_DELAY, "delay value too big")`


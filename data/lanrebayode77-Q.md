## Redundant Array
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L289
```
address[] memory memberlistWards = new address[](1);
```
Since the array length will be one, there is no need to declare the address variable as an array, best declared as an address type.
```
address memberlistWards = address(this);
```
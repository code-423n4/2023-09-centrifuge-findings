# Must implement Transfer Event in `transfer` and `transferFrom` functions (ERC-20)

In the [EIP-20](https://eips.ethereum.org/EIPS/eip-20) document, as a standard for ERC-20 tokens, the use of `event Transfer(address indexed _from, address indexed _to, uint256 _value)` is required to monitor or track the flow of the token. An example of its application is also [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9b3710465583284b8c4c5d2245749246bb2e0094/contracts/token/ERC20/ERC20.sol#L66). 

```solidity
File : src/token/Tranche.sol

60 : return super.transfer(to, value);

69 : return super.transferFrom(from, to, value);
```

[[60](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L60), [69](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L69)]
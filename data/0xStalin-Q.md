## [L-01] Using encodePacked with two consequtive dynamic types could cause hashes collisions
- There are multiple instances where encodePacked is used to hash values, and some of those values are two consequtive dynamic types
- From the solidity documentation: https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=collisions#non-standard-packed-mode 
   > If you use keccak256(abi.encodePacked(a, b)) and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of a into b and vice-versa. More specifically, abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c").
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L152-L160
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L210
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L274
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L323-L325
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L381
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L449
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L481
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L509
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L540
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L566
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L593-L595
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L622-L630
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L665-L673
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L742-L748
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L771
  - https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L790


**Fix:**
- As per the Solidity documentation, `Unless there is a compelling reason to use abi.encodePacked, abi.encode should be preferred.`

## [L-02] Not validating if the delay would be set less than a minimum acceptable time in the Root contract
- When [updating the delay time](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L43-L51) in the Root Contract, there is no check to validate that the new delay would be set to a minimum value, this means, that is possible to set it to 0.
  - If delay is set to 0, all DelayedAdmin wards will be able to rely to any address instanstaniously.
```solidity
function file(bytes32 what, uint256 data) external auth {
  if (what == "delay") {
    //@audit-ok => Only validating if the new delay doesn't exceed the upper limit
    require(data <= MAX_DELAY, "Root/delay-too-long");

    //@audit-issue => Not validating if the new delay is lower than a lower limit, it could be 0
    delay = data;
  } else {
    revert("Root/file-unrecognized-param");
  }
  emit File(what, data);
}
```
**Fix:**
- Check if the new delay is not lower than a valid acceptable range. (1 hour? 2 hours?)

## [L-03] Not validating if destinationChainId is a supported chain could lead to users getting their TrancheTokens burnt forever when transfering tokens from one EVM chain to another
- When users want to transfer their TrancheTokens from one chain to another they can use the [`PoolManager::transferTrancheTokensToEVM()`](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L149-L163) function, the problem is that there is not a check to validate if the inputed `destinationChainId` is a supported chain where the TrancheTokens could actually be minted.
- If the inputed `destinationChainId` is not a valid chain, the TrancheTokens from the user will still be burnt in the SourceChain, and a message to the Axelar Network will be sent, but as I mentioned, if the `destinationChainId` is not supported, those tokens won't be able to be minted, and in the Source Chain the TrancheTokens would be lost.
- Since this is caused due to an user error I'm submitting it as a Low Issue, but it would be very benefitial to prevent this problem from actually happening.

```solidity
function transferTrancheTokensToEVM(
    uint64 poolId,
    bytes16 trancheId,
    uint64 destinationChainId,
    address destinationAddress,
    uint128 amount
) public {
    TrancheTokenLike trancheToken = TrancheTokenLike(getTrancheToken(poolId, trancheId));
    require(address(trancheToken) != address(0), "PoolManager/unknown-token");
    
    //@audit-info => burns TrancheToken in the chain where this functions is called
    trancheToken.burn(msg.sender, amount);
    gateway.transferTrancheTokensToEVM(
        poolId, trancheId, msg.sender, destinationChainId, destinationAddress, amount
    );
}
```

**Fix:**
- Add a check to validate if the inputed `destinationChainId` is supported, and if not, revert the tx, don't let the function to burn the users tokens if the `destinationChainId` is not supported.

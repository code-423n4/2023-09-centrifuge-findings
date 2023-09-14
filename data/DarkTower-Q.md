###### Title: `RestrictionManager.updateMembers` doesn't implement `require(block.timestamp <= validUntil` check to verify if validUntil is greater than block.timestamp
**Description:** The function updateMemeber in RestrictionManager.sol doesn't implement the check `require(block.timestamp <= validUntil` which can lead to pass a validUntil value less than block.timestamp.
The similar function `updateMember` implements this check here: https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L57C4-L60C6

**Impact:** This may not immediately disrupt the contract's functionality, but it could lead to confusion, unexpected behavior, or a state where members have invalid access rights.
 
**Line of Code:** https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L62C4-L68C2
**Recommendation:**
Add this line of code in the function. 
```solidity
function updateMembers(address[] memory users, uint256 validUntil) public auth {
+    require(validUntil >= block.timestamp, "validUntil must be greater than or equal to the current block timestamp");
     // Rest of the function
}
```

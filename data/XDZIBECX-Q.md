## issue N°1 :

- https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Factory.sol#L114

`restrictionManager.updateMember(RootLike(root).escrow(), type(uint256).max);`
- details : 
This line is calling the updateMember function of the restrictionManager contract and passing two arguments:
- RootLike(root).escrow(): This part retrieves the escrow address from the root contract using the RootLike interface and calls the updateMember function with this address as an argument.
- type(uint256).max: This part seems to be an attempt to set the validUntil parameter to the maximum possible value, implying that the member's status should never expire.
so the updateMember function is responsible for updating the membership status of an address in the restrictionManager contract the problem is Setting the validUntil parameter to type(uint256).max effectively means that the membership status will never expire. While this might be intended, setting an expiration date to the maximum possible value could be a security risk. In some cases, it might be safer to set a reasonable expiration date to limit the duration of membership.
- recommendation:
set validUntil to the maximum possible value for all cases and  set reasonable expiration dates
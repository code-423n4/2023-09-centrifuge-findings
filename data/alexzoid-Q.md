## Optimizing the `ERC20` Implementation: Inheritance from the `Auth` Contract

- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L51-L65
- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Auth.sol


Instead of duplicating functions and modifiers, consider having the `ERC20` contract inherit from the `Auth` contract. This will automatically provide the `ERC20` contract with all the functionalities of the `Auth` contract without the need for duplication.

```solidity
contract ERC20 is Auth, Context { ... }
```

## Membership Management Limitations in the `RestrictionManager` Contract

- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L57-L60

There is no function to remove or invalidate a member's membership. While the `updateMember` function allows setting a new validity period for a member, it does not provide a mechanism to invalidate a member immediately. This is a potential oversight as there might be scenarios where a member needs to be removed or suspended immediately, without waiting for their membership to expire.

Consider implementing a `removeMember` function that sets the `validUntil` timestamp of a member to a past date, effectively invalidating their membership. 


## Functionality Gaps in the `TrancheToken` Contract: The Need for Token Burn Capabilities

- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol#L58-L74

The `TrancheToken` contract is designed as an extension of the standard `ERC20` token, with added functionalities from the `ERC1404` standard. Its primary goal is to implement transfer restrictions based on the rules defined in the associated `RestrictionManager`. While the contract provides functionalities for creating tokens (`mint`), it doesn't have an equivalent burn function to allow for the destruction of tokens.

Introduce a `burn` function in the `TrancheToken` contract that allows token holders or authorized addresses to destroy a specified amount of tokens. 
```solidity
function burn(address from, uint256 value) public override restricted(_msgSender(), from, value) {
    return super.burn(from, value);
}
```

## Leveraging Interface Inheritance Across Contracts

Inheritance from interface are missed across the contracts. When a contract inherits from an interface, it's mandated to provide implementations for all the methods declared in that interface. This ensures a strict adherence to a specific contract API. 

Create a distinct file for each interface, named appropriately. Implement inheritance from contract's interface, e.g.:

- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L14
```solidity
contract RestrictionManager is Auth, MemberlistLike {
```

- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L26
```solidity
contract LiquidityPoolFactory is Auth, LiquidityPoolFactoryLike {
```

- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L71
```solidity
contract TrancheTokenFactory is Auth, TrancheTokenFactoryLike {
```
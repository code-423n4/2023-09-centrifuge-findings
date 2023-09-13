## Misleading Title
```
/// @title  Delayed Admin
```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L7C24-L7C24

## Mitigations
The contract is PauseAdmin.sol, it should be Pause Admin instead of Delayed Admin

## Wrong Standard of script file name
For the `script/Deployer.sol` it should be `Deployer.s.sol`

# No checking of value == 0, thus wasting gas
No checking of value == 0 allows users to transfer 0 tokens. Should include a check to revert and refund remaining gas.
```
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L91C7-L91C7

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L106
```

## Duplicate Functions
Both functions acts similarly, there isn't a need for two functions. One will do, which also save deployment gas fees.
```
    function member(address user) public view {
        require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");
    }

    function hasMember(address user) public view returns (bool) {
        if (members[user] >= block.timestamp) {
            return true;
        }
        return false;
    }
```
# QA-01: Missing check in the constructor of `Root.sol` if the initially set delay is smaller than the `MAX_DELAY`

Link to code:
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34-L40

## Description:
There is no check in the constructor of `Root.sol` that makes sure that the initially set delay is smaller than the `MAX_DELAY`. This can lead to delays in the launch of the project in case the dalay is set to more than the `MAX_DELAY` since only wards of root can change this value and it would take more than `MAX_DELAY` to set a new ward for the root contract. Consider adding a check that ensures that the initially set delay is smaller than `MAX_DELAY` 

# Issue N1: Unused Events

## Description

Both the **DelayedAdmin** and **PauseAdmin** contracts contain an unused event called **File**. This event is declared but never emitted within either of the contracts. This leads to dead code and potential confusion for developers working with the codebase.

## Code Snippet

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/DelayedAdmin.sol#L16 and https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L19

## Remediation

Remove unused event, or move to base contract, to prevent unused code.

# Issue N2: Redundant Initialization

## Description

In the given code snippet, the paused state variable is explicitly initialized to false. In Solidity, state variables are automatically initialized to their default values, which for boolean types is false. Explicitly setting this variable to false is redundant and does not add any functional value to the code. It also unnecessarily increases the bytecode size slightly, leading to a minor increase in deployment costs.

## Code Snippet

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L23

## Remediation

Remove the explicit initialization of the paused variable. The Solidity compiler will automatically initialize it to false.

# Issue N3: Missing Check on delay in the constructor

## Description

The constructor accepts a delay parameter but doesn't check its validity or constraints.

## Code Snippet

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34

## Remediation

Add a check in the constructor to ensure that the passed delay is within acceptable bounds.
> require(_delay <= MAX_DELAY, "Root/delay-too-long");
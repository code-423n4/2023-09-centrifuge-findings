# Issue N1: Unused Events

## Description

Both the **DelayedAdmin** and **PauseAdmin** contracts contain an unused event called **File**. This event is declared but never emitted within either of the contracts. This leads to dead code and potential confusion for developers working with the codebase.

## Code Snippet

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/DelayedAdmin.sol#L16 and https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L19

## Remediation

All declared events should be used within the contract or removed to avoid dead code, also since both **DelayedAdmin** and **PauseAdmin** inherit from **Auth**, moving the event to **Auth** makes it available to any contract that inherits from **Auth**, which could be a useful optimization if the event is intended for broad use. Additional note that **Root** also has event **File** and inherits from **Auth**, so if this event isn't redundant can be removed to Auth for this contract too.
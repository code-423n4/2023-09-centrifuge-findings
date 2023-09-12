https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L46

line 46 in UserEscrow.sol can be wrapped with unchecked{} because the require in line 37:
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L37

is checking that destinations[token][destination] >= amount
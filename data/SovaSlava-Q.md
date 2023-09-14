Q1 - Constructor dont emit events AddIncomingRouter() and UpdateOutgoingRouter()
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L102
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L103

Q2 - Changing "delay" variable should be delayed by current value of this variable. Not immediately
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L46

Q3 - There is not list of allowed chains, which this project support. User can make mistake, when transfer tokens, using PoolManager.transferTrancheTokensToEVM() and lost tokens
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L161

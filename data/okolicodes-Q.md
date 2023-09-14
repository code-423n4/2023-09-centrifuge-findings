## Title:
Failure to add additional Validation checks as suggested by the Docs

## Vulnerabilty details
 The  transferOut function in the UserEscrow.sol contract fails to implement a require or if Statement as suggested by the docs for additional protection. Although this function is heavily guarded by the auth modifier but the additional protection was not implemented
Take a look at the implemented code below 
 https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L36C5-L49C51  
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L43C10-L44C51

## Recommendation:
Implement a require check
  require receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max),
            "UserEscrow/receiver-has-no-allowance"
1) RestrictionManager:
   In the updateMembers() functions, the validUntil is not validated.
   
   ```
       function updateMembers(address[] memory users, uint256 validUntil) public auth { // require is 
       missing for validUntil, meaning can add members who are now disqualified
        uint256 userLength = users.length;
        for (uint256 i = 0; i < userLength; i++) {// no check for zero address
            updateMember(users[i], validUntil);
        }
    }
   ```


2) Root:
   The value passed in the constructor of delay could be larger than max. There is no validation.
   Also, there is no validation for the escrow address which is immutable. The input params should be checked.


3) LiquidityPool:
   The interfaces declared are not inherited to implementation contract. Example, InvestmentManagerLike is not inherited into the LiquidityPool contract.

4) InvestmentManager:
   Force typecasting of uint128 to uint256 can result in loss of value due to precision.
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
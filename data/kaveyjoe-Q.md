## 1 .  Lack of Permission Validation in transferOut

In the transferOut function, there is a condition that allows the transfer if either the receiver is the destination address or if the receiver has received the full allowance from the destination address. This may lead to potential unauthorized transfers.

**Code snippet:**

require(
    receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max),
    "UserEscrow/receiver-has-no-allowance"
);

**Recommendation**

onsider adding an additional check to ensure that the caller of the function is either the destination address or an authorized admin. 

require(
    msg.sender == destination || wards[msg.sender] == 1,
    "UserEscrow/unauthorized-access"
);



## 2 . Authorization check 

In the approve function, there is a call to SafeTransferLib.safeApprove to approve tokens. This is intended to be a secure way to approve token transfers. However this  check whether the spender has the authority to withdraw the approved funds.


function approve(address token, address spender, uint256 value) external auth {
    SafeTransferLib.safeApprove(token, spender, value);
    emit Approve(token, spender, value);
}


Issue: Lack of authorization check to ensure that only authorized users (wards) can approve funds. Any external account can call this function and potentially approve funds to themselves or other unauthorized addresses.


## 3 . Lack of Access Control in _newRestrictionManager

In the _newRestrictionManager function of the TrancheTokenFactory contract, there is a potential security vulnerability due to the lack of proper access control. Specifically, the updateMember function is called on a new instance of RestrictionManager without verifying the caller's authorization. This could allow unauthorized users to update members, which is a critical security risk.

**Recommendation**
To mitigate this issue, it is essential to implement access control checks before calling the updateMember function within the _newRestrictionManager function. Ensure that only authorized users have the permission to update members.

**Code snippet**:

function _newRestrictionManager(address[] calldata restrictionManagerWards) internal returns (address memberList) {
    RestrictionManager restrictionManager = new RestrictionManager();

    // Access control check needed before calling updateMember
    require(msg.sender == root, "Unauthorized access");

    restrictionManager.updateMember(RootLike(root).escrow(), type(uint256).max);

    restrictionManager.rely(root);
    for (uint256 i = 0; i < restrictionManagerWards.length; i++) {
        restrictionManager.rely(restrictionManagerWards[i]);
    }
    restrictionManager.deny(address(this));

    return (address(restrictionManager));
}


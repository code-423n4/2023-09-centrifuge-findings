# Low

### Factory.sol: No zero length check for wards 

If the wards array is empty, the LP will permanently have no wards and will need to be reimplemented in the future in order to be pausable. Worst case scenario, the protocol isn't able to address this issue before the LP gets more traction. Consider adding a fix similar to the following:

```
    require(awards.length != 0, "revert string");
```

### Escrow.sol: Transactions from/to it will stop working if a ward modifies the approval

The approval function below can be found in the escrow contract:
```
    function approve(address token, address spender, uint256 value) external auth { 
        SafeTransferLib.safeApprove(token, spender, value); 
        emit Approve(token, spender, value);
    }
```
The issue here is everyt ime `safeApprove()` is called with a value below `typeof(uint256).max`, any further transactions that does not decrease its allowance to zero will revert. For example, everytime `handleTransfer()` is called in the pool manager, its allowance is increased, and the transfer takes place, which instantly decreases it. If its allowance is higher than zero, every call to this function will fail.

### Root.sol: Scheduled rely cannot be executed on current time

In `executeScheduledRely()`, the following check is made in order to guarantee it's time of execution is right.

```
        require(schedule[target] < block.timestamp, "Root/target-not-ready");
```
The line above is the validation check of the function. Notice that if the scheduled time is exact, the transaction will revert. This could happen in L2s such as arbitrum where there are multiple blocks every second. 

To fix this, consider rewriting as followed:
```
        require(schedule[target] <= block.timestamp, "Root/target-not-ready");
```

# Non Critical

### Auth.sol: Consider adding a check to prevent the last ward from calling deny()

If the contract deployer calls `deny()` on itself, the pausing functions are lost, as there are no more wards to execute their functions.

### ERC20.sol: Consider changing the contract's `version`

Version 3 is the current version of the Dai stablecoin, which is the basis for the contract. This variable should be reconsidered as it is the first interaction in this contract.

### Root.sol: Consider adding a check to see whether the rely address is a contract

The function is intended to assign values to contracts only. The protocol may want to consider adding a check to see if the byte code size is higher than zero to guarantee the address is in fact a contract.
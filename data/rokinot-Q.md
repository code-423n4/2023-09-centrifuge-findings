# Low

### Factory.sol: No zero length check for wards 

If the wards array is empty, the LP will permanently have no wards and will need to be reimplemented in the future in order to be pausable. Worst case scenario, the protocol isn't able to address this issue before the LP gets more traction. Consider adding a fix similar to the following:

```
    require(awards.length != 0, "revert string");
```

### InvestmentManager.sol: Pools are not fully ERC4626 compliant

According to [EIP-4626](https://eips.ethereum.org/EIPS/eip-4626), functions with the purpose of calculating how many tokens someone will earn by withdrawing shares and how many shares someone earns by depositing X tokens should be rounded up, however the protocol is using the same logic as `previewDeposit` and `previewRedeem`, which is to round down.

In all functions, `_calculateCurrencyAmount()` is called, with the following lines of code:

```
uint256 currencyAmountInPriceDecimals = _toPriceDecimals(
            trancheTokenAmount, trancheTokenDecimals, liquidityPool
        ).mulDiv(price, 10 ** PRICE_DECIMALS, MathLib.Rounding.Down);
```
As a result, all calculations are rounded down. In order to fix this, create a conditional that allows you to choose whether the function rounds up or down.

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

### Factory.sol: Token address shouldn't be assumed to be equal in all evm chains

Certain EVM chains, like zkSync, can use a different formula for derivating the address of a deployed contract, meaning if the project decides to deploy on said chain, the comment [here](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Factory.sol#L93) will be inaccurate.

For more information: [zkSync docs](https://era.zksync.io/docs/reference/architecture/differences-with-ethereum.html#create-create2)

Moreover, `msg.sender` will be a different address if a transaction to deploy the tranche token is sent to a sequencer on an optimistic rollup, from the L1 as mentioned in my other issue.


For more information: [Arbitrum docs](https://docs.arbitrum.io/arbitrum-ethereum-differences#l1-to-l2-messages)

### Auth.sol: Consider adding a check to prevent the last ward from calling deny()

If the contract deployer calls `deny()` on itself, the pausing functions are lost, as there are no more wards to execute their functions.

### ERC20.sol: Consider changing the contract's `version`

Version 3 is the current version of the Dai stablecoin, which is the basis for the contract. This variable should be reconsidered as it is the first interaction in this contract.

### Root.sol: Consider adding a check to see whether the rely address is a contract

The function is intended to assign values to contracts only. The protocol may want to consider adding a check to see if the byte code size is higher than zero to guarantee the address is in fact a contract.

## Make sure to always compile with the same flags as this contest

Right now, in the `foundry.toml` file, we can see the project is using the following flags:

```
solc_version = "0.8.21"   # Override for the solc version (setting this ignores `auto_detect_solc`)
# evm_version = "paris" # to prevent usage of PUSH0, which is not supported on all chains
```

This shows us the protocol is aware of breaking changes coming from solidity v0.8.21, and the shangai update as well. As a consequence, in the future, these flags must continue to be used as well if the protocol does not plan to go through another audit. 
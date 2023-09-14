Description:

In the `InvestmentManager` contract, there is a vulnerability related to the shadowing of state variables within the `_isAllowedToInvest` internal function. This vulnerability, while not critically severe, can lead to code confusion and potential bugs.

## Impact
Detailed description of the impact of this finding.

## Proof of Concept

Shadowing of State Variables In the [_isAllowedToInvest](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L651-L662) internal function, the parameter name currency is shadowing the state variable currency defined in the function's scope. This can make the code less clear and potentially lead to errors.

```solidity
function _isAllowedToInvest(uint64 poolId, bytes16 trancheId, address currency, address user)
    internal
    returns (bool)
{
    address liquidityPool = poolManager.getLiquidityPool(poolId, trancheId, currency);
    // ...
}
```

In this code, the function parameter `currency` shares the same name as a state variable defined elsewhere in the contract. The parameter `currency` effectively shadows the state variable, making it unclear which variable is being referred to within the function.

**Potential Incorrect Behavior:**

Due to the shadowing of the state variable, if there are unintentional references to `currency` within the `_isAllowedToInvest` function without explicitly specifying the parameter, it will mistakenly refer to the function parameter instead of the intended state variable. This can result in incorrect behavior if the state variable and parameter have different values.

Example:

Assuming the state variable `currency` is set to address `0x1111111111111111111111111111111111111111` in the contract constructor, consider the following code snippet:

```solidity
function _isAllowedToInvest(uint64 poolId, bytes16 trancheId, address currency, address user)
    internal
    returns (bool)
{
    address liquidityPool = poolManager.getLiquidityPool(poolId, trancheId, currency);
    
    // Incorrect usage (intentionally without specifying parameter)
    if (currency == 0x1111111111111111111111111111111111111111) {
        // This condition will behave incorrectly because 'currency' refers to the parameter,
        // and it will always evaluate to true regardless of the state variable's value.
    }
    
    // ...
}
```

In this example, the condition `currency == 0x1111111111111111111111111111111111111111` is intended to check the equality of the state variable `currency` with a specific address. However, due to the shadowing, it mistakenly refers to the function parameter `currency`, leading to incorrect behavior.

## Tools Used
Vs

## Recommended Mitigation Steps

Use distinct names for function parameters and state variables to avoid shadowing.

```solidity
function _isAllowedToInvest(uint64 poolId, bytes16 trancheId, address currencyAddress, address user)
    internal
    returns (bool)
{
+   address liquidityPool = poolManager.getLiquidityPool(poolId, trancheId, currencyAddress);
-   address liquidityPool = poolManager.getLiquidityPool(poolId, trancheId, currency);
    // ...
}
```
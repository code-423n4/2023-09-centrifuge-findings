# L-01 : `totalAssets` function cannot be called by anyone from `InvestmentManger.sol` 
`totalAssets` function  looks like this : this is callable by anyone .

```solidity 
 function totalAssets(uint256 totalSupply, address liquidityPool) public view returns (uint256 _totalAssets) {
        _totalAssets = convertToAssets(totalSupply, liquidityPool); 
    }
```
but , 
```solidity 
 function convertToAssets(uint256 _shares, address liquidityPool) public view auth returns (uint256 assets)
```
This is a view function to check the amount of assets from certain amount of shares . Problem is the `auth` modifier . this doesnot let anyone to call the function . This should be removed to make `totalAssets` function  work properly. Otherwise ,  only wards can call this function . 
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L319
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L338
# L-02 : `convertToShares` function should be called by anyone from `InvestmentManger.sol` 

The `convertToShares` function is to check the amount of shares for certain amount of asset . This is a view function and should be callable by everyone to check their asset to share conversion . But because  of `auth ` modifier the function caller requires to  be an ward . Remove this to get the function work  properly . 
```solidity 
 function convertToShares(uint256 _assets, address liquidityPool) public view auth returns (uint256 shares)
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L325

## [NC-1] The `ApproveLike` interface in the `Escrow.sol` file is not used.
File: https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Escrow.sol#L7

Consider deleting the unused interface.

## [NC-2] Some fields in some struct are not commented
Consider adding comments for each struct field to enhance readability and accesibility of the codebase.
1. All fields in the `Pool` struct are not commented.
```
struct Pool {//@audit-info uncommented fields of a struct
    uint64 poolId;
    uint256 createdAt;
    mapping(bytes16 => Tranche) tranches;
    mapping(address => bool) allowedCurrencies;
}
```
File: https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L53-L58

2. Some fields on this structs are not commented.
```
struct Tranche {
    address token;//@audit-info comment struct.
    uint64 poolId;
    bytes16 trancheId;
    // important: the decimals of the leading pool currency. Liquidity Pool shares have to be denomatimated with the same precision.
    uint8 decimals;
    uint256 createdAt;
    string tokenName;
    string tokenSymbol;
    /// @dev Each tranche can have multiple liquidity pools deployed,
    /// each linked to a unique investment currency (asset)
    mapping(address => address) liquidityPools; // currency -> liquidity pool address
}
```
File: https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol#L61-L73

## [NC-3] Address of the pauser should be included in the `Pause()` and `Unpause()` events.
The `Pause()` and `Unpause()` events does not include information of the person who paused the system. Consider adding the address of the pause to the `Pause()` and `Unpause()` events.
File: https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L27-L28
```
event Pause();//@audit add address of pauser.
    event Unpause();
```
## [NC-4] Commented code may break the code
commented code may mean unfinished work sometimes. 

File: https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L149

```
function mint(uint256 shares, address receiver) public returns (uint256 assets) {
        // require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");//@audit-info commented code.
        assets = investmentManager.processMint(receiver, shares);
        emit Deposit(address(this), receiver, assets, shares);
    }
```
Commented codes should be corrected.

## [NC-5] Wrong title comment in the PausableAdmin contract.
The PausableAdmin contract title should have "Pausable Admin" not "Delayed Admin". 

File: https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L7
```
/// @title  Delayed Admin @audit
/// @dev    Any ward can manage accounts who can pause.
///         Any pauser can instantaneously pause the Root.
contract PauseAdmin is Auth {
    Root public immutable root;
...
```

## [NC-6] ERC20.sol is supposed to inherit from Auth.sol instead of rewritting all the implementation in Auth.sol in the ERC20.sol contract.

ERC20.sol has all the state, events and functions in the Auth.sol contract. Consider making ERC20.sol inherit from Auth.sol to reduce code duplications.

Files: 
- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L17-L65
- https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Auth.sol#L7

## [NC-7] Variable not used 
1. The parameter `liquidityPool` in the InvestmentManager.sol#_toPriceDecimals() function is not used.
File: https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L676
2. The `_price` local variable in the Gateway.sol#handle() function is not used.

File: https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L302
Variables that are not used should be deleted.

## [L-1] `wards` parameter of the `newLiquidityPool()` function shadows Auth.sol.wards state variable.

`wards` parameter of the `newLiquidityPool()` function shadows Auth.sol.wards state variable. Consider renaming the parameter to avoid variable shadowing.

File: https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Factory.sol#L48

```
contract LiquidityPoolFactory is Auth {
    address immutable root;

    constructor(address _root) {
        root = _root;

        wards[msg.sender] = 1;
        emit Rely(msg.sender);
    }

    function newLiquidityPool(
        uint64 poolId,
        bytes16 trancheId,
        address currency,
        address trancheToken,
        address investmentManager,
        address[] calldata wards
    ) public auth returns (address) {
        LiquidityPool liquidityPool = new LiquidityPool(poolId, trancheId, currency, trancheToken, investmentManager);

        liquidityPool.rely(root);
        for (uint256 i = 0; i < wards.length; i++) {//@audit variable shadowing wards.
            liquidityPool.rely(wards[i]);
        }
...
```
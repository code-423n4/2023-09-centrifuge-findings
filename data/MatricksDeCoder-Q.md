### NC-1 - Overly complex Modifiers

Modifiers should be used to only make straightforward checks for conditions. The following modifiers are overly complicated

[Tranche.sol line 35](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L35)
```solidity
  modifier restricted(address from, address to, uint256 value) {
        uint8 restrictionCode = detectTransferRestriction(from, to, value);
        require(restrictionCode == restrictionManager.SUCCESS_CODE(), messageForTransferRestriction(restrictionCode));
        _;
    }
```
Modifier has 2 other functions within it detectTransferRestriction(from, to, value) and messageForTransferRestriction(restrictionCode) 

[Router.sol lines43](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/routers/axelar/Router.sol#L43)
```
modifier onlyCentrifugeChainOrigin(string calldata sourceChain, string calldata sourceAddress) {
        require(msg.sender == address(axelarGateway), "AxelarRouter/invalid-origin");
        require(
            keccak256(bytes(axelarCentrifugeChainId)) == keccak256(bytes(sourceChain)),
            "AxelarRouter/invalid-source-chain"
        );
        require(
            keccak256(bytes(axelarCentrifugeChainAddress)) == keccak256(bytes(sourceAddress)),
            "AxelarRouter/invalid-source-address"
        );
        _;
    }
```
Modifier has 3 require checks statements 


**Impact**-> It affects the code  maintainability, readability, auditability, can introduce errors, result in code complexity or even cause unintended effects in code.

**Recommendation** -> It is recommended modifiers be kept simple, complex logic that is long, uses other functions, and may have side effects can be put into internal functions instead


### NC-2 - Better Modifiers naming

Modifiers should be named in ways to show what they constrain, allow, disallow etc in terms of familiar naming like starting with only.., when.., is... not.., while, can.. etc. The following modifiers can be named better

[modifier withApproval(address owner) LiquidityPool.sol line 97](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L97C5-L97C43) => modifier onlyApproved

[modifier restricted(address from, address to, uint256 value) Tranche.sol line 35](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L35) => notRestricted ; as its enforcing no transfers for restricted cases 

[  modifier auth() ERC20.sol line 51](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L51C3-L51C22) => onlyAuthorized or onlyAuth 

[  modifier auth() Auth.sol line 26 ](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Auth.sol#L26)

[modifier pauseable() Gateway.sol line 124](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L124C5-L124C27) => onlyRootNotPaused() to add more clarity that functions attached with pauseable only work when RootLike is not paused 


**Impact**-> It affects the code  maintainability, readability, auditability, can introduce errors, in application of modifier by devs if intention is not understood e.g pauseable may be interpreted as when root is paused instead of not paused and not applied to functions 

**Recommendation** -> It is recommended modifiers be named appropriately and clearly as discussed above


### NC-3 - Add require strings into variables for reuse

Require error strings can be added into variables that can be resued and also create cleaner code and avoid code duplication and or errors. 

e.g in ERC20.sol 
[require(allowed >= subtractedValue, "ERC20/insufficient-allowance") ERC20.sol line 150](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L150C9-L150C77) error string is duplicated below but could be saved in variable e.g string insuffAllowance = "ERC20/insufficient-allowance"

[require(allowed >= value, "ERC20/insufficient-allowance") ERC20.sol line 179](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L179)

=> require(allowed >= value, insuffAllowance)

[require(currencyPayout != 0, "InvestmentManager/zero-payout") InvestmentManager.sol#L284](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L284C9-L284C71) is resued in same contract below

[ require(trancheTokenPayout != 0, "InvestmentManager/zero-payout"); InvestmentManager.sol#L301](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L301) whereas "InvestmentManager/zero-payout" could have been put in variable

The above are a few examples to show problem

Beside above duplication case, generally all other require error strings may be best cleaner in own variables. We see this pattern in the modifer below; where message is messageForTransferRestriction(restrictionCode) is in variable/function for cleaner usage

[Tranche.sol line 35](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L35)
```solidity
  modifier restricted(address from, address to, uint256 value) {
        uint8 restrictionCode = detectTransferRestriction(from, to, value);
        require(restrictionCode == restrictionManager.SUCCESS_CODE(), messageForTransferRestriction(restrictionCode));
        _;
    }
```

**Impact**-> It affects the code  maintainability, readability, auditability, can introduce errors, in reusing strings but make typos etc 

**Recommendation** -> It is recommended require error strings be saved into variables and reused as appropriate in the specific require instance. 

### NC-4 - Inconsistent underscore in constructors 

```
//In some constructors the arguments/parameters 
 _variableName //underscore is pre 
variableName_  //whereas in others it is post e.g  
```

All other contract constructor e.g src/LiquidityPool.sol; src/InvestmentManager.sol; src/PoolManager.sol; src/admins/PauseAdmin.sol; src/admins/DelayedAdmin.sol; src/token/Tranche.sol; src/token/ERC20.sol; src/gateway/Gateway.sol; src/gateway/routers/axelar/Router.sol;   use trailing underscore as in example below

```
constructor(uint64 poolId_, bytes16 trancheId_, address asset_, address share_, address investmentManager_)
```

[whereas e.g the following use preceding underscore Root.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L34)
```
constructor(address _escrow, uint256 _delay) // underscore is leading 
```
[Factory.sol line 29 _root](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Factory.sol#L29C5-L29C33)
```
constructor(address _root) {
```

**Impact**-> It affects the code  maintainability, readability, auditability, and inconsistency 

**Recommendation** -> It is recommended to be consistent in implementation of underscore to constructor variables 

### NC-5 - Solidity version too recent 

Contracts make use of latest soliity version 0.8.21 

**Impact**-> It may lead to errors, bugs, miscompilation, etc as latest version may have bugs, errors, problems, quirks, challenges that have not yet been uncovered. 0.8.20 may not work well or hinder code determinism on platforms like Arbitrum due to introduction of PUSH0 etc 

**Recommendation** -> It is recommended to use maybe 0.8.20 or 0.8.19

### NC-6 - Return or revert early in functions 

It is recommended for functions with checks that revert to revert early or with return logic or with other simple if else logic to start with the simple logic or return checks to avoid running complicated logic if conditions check. 

Example instances are below but advised to recheck all code and refactor appropriately for examples that may not be listed in the below 


```
// --- Administration --- InvestmentManager.sol line 103
    function file(bytes32 what, address data) external auth {
        if (what == "gateway") gateway = GatewayLike(data);
        else if (what == "poolManager") poolManager = PoolManagerLike(data);
        else revert("InvestmentManager/file-unrecognized-param");
        emit File(what, data);
    }

// Better as below 
function file(bytes32 what, address data) external auth {
        if (what != "gateway" || what != "poolManager") revert("InvestmentManager/file-unrecognized-param");
        if( what == "gateway") gateway = GatewayLike(data);
        else poolManager = PoolManagerLike(data);
        emit File(what, data);
    }
```

[InvestmentManager.sol from line 117 ](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L117C5-L141C6)
```
function requestDeposit(uint256 currencyAmount, address user) public auth {
        address liquidityPool = msg.sender;
        LiquidityPoolLike lPool = LiquidityPoolLike(liquidityPool);
        address currency = lPool.asset();
        uint128 _currencyAmount = _toUint128(currencyAmount);

        // Check if liquidity pool currency is supported by the Centrifuge pool
        poolManager.isAllowedAsPoolCurrency(lPool.poolId(), currency);
        // Check if user is allowed to hold the restricted tranche tokens
        _isAllowedToInvest(lPool.poolId(), lPool.trancheId(), currency, user);
        if (_currencyAmount == 0) {
            // Case: outstanding investment orders only needed to be cancelled
            gateway.cancelInvestOrder(
                lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset())
            );
            return;
        }

        // Transfer the currency amount from user to escrow. (lock currency in escrow).
        SafeTransferLib.safeTransferFrom(currency, user, address(escrow), _currencyAmount);

        gateway.increaseInvestOrder(
            lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset()), _currencyAmount
        );
    }

/// Better as below start with the simple if check that returns 
function requestDeposit(uint256 currencyAmount, address user) public auth {
        uint128 _currencyAmount = _toUint128(currencyAmount);
        // start with if check that returns 
        if (_currencyAmount == 0) {
            // Case: outstanding investment orders only needed to be cancelled
            gateway.cancelInvestOrder(
                lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset())
            );
            return;
        }
        address liquidityPool = msg.sender;
        LiquidityPoolLike lPool = LiquidityPoolLike(liquidityPool);
        address currency = lPool.asset();
        

        // Check if liquidity pool currency is supported by the Centrifuge pool
        poolManager.isAllowedAsPoolCurrency(lPool.poolId(), currency);
        // Check if user is allowed to hold the restricted tranche tokens
        _isAllowedToInvest(lPool.poolId(), lPool.trancheId(), currency, user);

        // Transfer the currency amount from user to escrow. (lock currency in escrow).
        SafeTransferLib.safeTransferFrom(currency, user, address(escrow), _currencyAmount);

        gateway.increaseInvestOrder(
            lPool.poolId(), lPool.trancheId(), user, poolManager.currencyAddressToId(lPool.asset()), _currencyAmount
        );
    }

```
Above are just examples of refactoring to start with simpler logic, branches, reverts, checks etc before the complex logic that may not run. It is important to go through all functions and not miss other related examples. Some other examples are below 

Root.sol line 43
```
  function file(bytes32 what, uint256 data) external auth {
        if (what == "delay") {
            require(data <= MAX_DELAY, "Root/delay-too-long");
            delay = data;
        } else {
            revert("Root/file-unrecognized-param");
        }
        emit File(what, data);
    }
\\ Better as below 
 function file(bytes32 what, uint256 data) external auth {
        if (what != "delay") revert("Root/file-unrecognized-param");
        require(data <= MAX_DELAY, "Root/delay-too-long");
        delay = data;
        emit File(what, data);
    }

```
etc etc

**Impact**-> No only does reverting early save gas it ensures such checks may no be forgotten if implemented last, it also improves code readability and auditability  

**Recommendation** -> It is recommended to revert early, return early, start with simpler conditions checks, start with least complex branch checks etc 

### NC-7 - function names should be camelCase 

Solidity functions should be camelCase 
[function SUCCESS_CODE() public view returns (uint8)](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L88)

[function DOMAIN_SEPARATOR() external view returns (bytes32)](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L79)

**Impact**-> It is best to follow best practise, style guide to avoid confusion, help with readability, maintainability and audittability 

**Recommendation** -> It is recommended change all instances not conforming to camelCase for functions

### NC-8 - contracts named like ERC although not fully like the namesake

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L23

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L12


Whilst others ar okay to name EscrowLike, AuthLike etc naming using ERC20Like, ERC20PermitLike, ERC1404Like etc may cause confusion that e.g ERC20Permit version was intended 

**Impact**-> It may lead to readability, auditability, problems with tooling, maintainability issues 

**Recommendation** -> It is recommended to not name contracts to be Like ERC counterparts even if they implement a few of the functionality or mimic them as this may create confusion that the real ERC contract was being implied. Rather name them differently or use the proper ERC contract and or interface etc 
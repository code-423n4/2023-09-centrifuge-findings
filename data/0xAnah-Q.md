# CENTRIFUGE NON-CRITICAL FINDINGS


## TABLE OF FINDINGS

| Number |Issue|Instances|
|-|:-|:-:|
| G-01| else-block not required | 1 |
| G-02| A modifier used only once and not being inherited should be inlined | 2 |
| G-03| Underscore (_) preceding private and internal state variable names | 1 |
| G-04| Use constants for state variables whose value is known beforehand and is never changed | 1 | 
| G-05| State variable shadowed by function parametert | 1 |



## [G-01] else-block not required
One level of nesting can be removed by not having an else block when the if-block returns

### 1 Instance
- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L669
```solidity
file: src/InvestmentManager.sol

666:    function _toUint128(uint256 _value) internal pure returns (uint128 value) {
667:        if (_value > type(uint128).max) {
668:            revert("InvestmentManager/uint128-overflow");
669:        } else {    @audit else block not required
670:            value = uint128(_value);
671:        }
672:    }
```
The code should be refactored to:
```diff
diff --git a/src/InvestmentManager.sol b/src/InvestmentManager.sol              
index fe94965..e1200db 100644                                                   
--- a/src/InvestmentManager.sol                                                 
+++ b/src/InvestmentManager.sol                                                 
@@ -666,9 +666,10 @@ contract InvestmentManager is Auth {                       
     function _toUint128(uint256 _value) internal pure returns (uint128 value) {
         if (_value > type(uint128).max) {                                      
             revert("InvestmentManager/uint128-overflow");                      
-        } else {                                                               
-            value = uint128(_value);                                           
-        }                                                                      
+        }                                                                      
+                                                                               
+        value = uint128(_value);                                               
+                                                                               
     }                                                                          
```


## [G-02] A modifier used only once and not being inherited should be inlined.

### 2 Instance
- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/PauseAdmin.sol#L28
```solidity
file: src/admins/PauseAdmin.sol

28:    modifier canPause() {
29:        require(pausers[msg.sender] == 1, "PauseAdmin/not-authorized-to-pause");
30:        _;
31:    }
```
Since the `canPause()` modifier in the `PauseAdmin` contract is only used in one function the `pause()` function and the contract is not inherited from we can a refactor the code and write the require statement in the `pause()` and discard the `canPause()` modifier. The diff below shows how the could be refactored:
```diff
diff --git a/src/admins/PauseAdmin.sol b/src/admins/PauseAdmin.sol                
index dc5d26e..fed1e7f 100644                                                     
--- a/src/admins/PauseAdmin.sol                                                   
+++ b/src/admins/PauseAdmin.sol                                                   
@@ -25,11 +25,6 @@ contract PauseAdmin is Auth {                                  
         emit Rely(msg.sender);                                                   
     }                                                                            
                                                                                  
-    modifier canPause() {                                                        
-        require(pausers[msg.sender] == 1, "PauseAdmin/not-authorized-to-pause"); 
-        _;                                                                       
-    }                                                                            
-                                                                                 
     // --- Administration ---                                                    
     function addPauser(address user) external auth {                             
         pausers[user] = 1;                                                       
@@ -42,7 +37,8 @@ contract PauseAdmin is Auth {                                   
     }                                                                            
                                                                                  
     // --- Admin actions ---                                                     
-    function pause() public canPause {                                           
+    function pause() public {                                                    
+        require(pausers[msg.sender] == 1, "PauseAdmin/not-authorized-to-pause"); 
         root.pause();                                                            
     }                                                                            
 }                                                                                
```

- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/routers/axelar/Router.sol#L56
```solidity
file: src/gateway/routers/axelar/Router.sol

56:    modifier onlyGateway() {
57:        require(msg.sender == address(gateway), "AxelarRouter/only-gateway-allowed-to-call");
58:        _;
59:    }
```
Since the `onlyGateway()` modifier in the `AxelarRouter` contract is only used in one function the `send()` function and the contract is not inherited from we can a refactor the code and write the require statement in the `send()` and discard the `onlyGateway()` modifier. The diff below shows how the could be refactored:
```diff
diff --git a/src/gateway/routers/axelar/Router.sol b/src/gateway/routers/axelar/Router.sol
index 58b0bba..671f224 100644
--- a/src/gateway/routers/axelar/Router.sol
+++ b/src/gateway/routers/axelar/Router.sol
@@ -53,10 +53,6 @@ contract AxelarRouter is Auth {
         _;
     }

-    modifier onlyGateway() {
-        require(msg.sender == address(gateway), "AxelarRouter/only-gateway-allowed-to-call");
-        _;
-    }

     // --- Administration ---
     function file(bytes32 what, address data) external auth {
@@ -86,7 +82,8 @@ contract AxelarRouter is Auth {
     }

     // --- Outgoing ---
-    function send(bytes calldata message) public onlyGateway {
+    function send(bytes calldata message) public {
+        require(msg.sender == address(gateway), "AxelarRouter/only-gateway-allowed-to-call");
         axelarGateway.callContract(axelarCentrifugeChainId, centrifugeGatewayPrecompileAddress, message);
     }
 }
```


## [G-03] Underscore (_) preceding private and internal state variable names
Private and internal state variables should have a preceding _ in their name unless they are constants.\
Reference: [Solidity documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

### 1 Instances
- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L17
```solidity
file: src/UserEscrow.sol

17:    mapping(address => mapping(address => uint256)) destinations;
```
The declaration should be refactored to:
```diff
diff --git a/src/UserEscrow.sol b/src/UserEscrow.sol
index 458cb01..c1bab10 100644
--- a/src/UserEscrow.sol
+++ b/src/UserEscrow.sol
@@ -14,7 +14,7 @@ interface ERC20Like {
 ///         transferred out to the pre-chosen destinations, by wards.
 contract UserEscrow is Auth {
     /// @dev Map by token and destination
-    mapping(address => mapping(address => uint256)) destinations;
+    mapping(address => mapping(address => uint256)) _destinations;

     // --- Events ---
     event TransferIn(address indexed token, address indexed source, address indexed destination, uint256 amount);
@@ -27,14 +27,14 @@ contract UserEscrow is Auth {

     // --- Token transfers ---
     function transferIn(address token, address source, address destination, uint256 amount) external auth {
-        destinations[token][destination] += amount;
+        _destinations[token][destination] += amount;

         SafeTransferLib.safeTransferFrom(token, source, address(this), amount);
         emit TransferIn(token, source, destination, amount);
     }

     function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
-        require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
+        require(_destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
         require(
             /// @dev transferOut can only be initiated by the destination address or an authorized admin.
             ///      The check is just an additional protection to secure destination funds in case of compromized auth.
@@ -43,7 +43,7 @@ contract UserEscrow is Auth {
             receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max),
             "UserEscrow/receiver-has-no-allowance"
         );
-        destinations[token][destination] -= amount;
+        _destinations[token][destination] -= amount;

         SafeTransferLib.safeTransfer(token, receiver, amount);
         emit TransferOut(token, receiver, amount);
```


## [G-04] Use constants for state variables whose value is known beforehand and is never changed
When you declare a state variable as a constant, you're telling the Solidity compiler that this variable's value is known at compile time and will never change during the contract's lifetime. Because of this:
The value of the constant state variable is computed and set at the time of contract deployment, not during runtime.

Since the value is known beforehand and cannot change, there is no need to store this value on the Ethereum blockchain, as it can be computed directly whenever needed.
When you access the constant state variable in your contract functions, the value is readily available (saved in the contracts bytecode) and does not require any computation.


### 1 Instance
- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L17
```solidity
file: src/Root.sol

17:    uint256 private MAX_DELAY = 4 weeks; @audit make constant
```
The value of the `MAX_DELAY` state variable is never change in the contract so it safe to make it a constant
```diff
diff --git a/src/Root.sol b/src/Root.sol
index 19ab79c..052a347 100644
--- a/src/Root.sol
+++ b/src/Root.sol
@@ -14,7 +14,7 @@ interface AuthLike {
 ///         is restricted to the timelock set by the delay.
 contract Root is Auth {
     /// @dev To prevent filing a delay that would block any updates indefinitely
-    uint256 private MAX_DELAY = 4 weeks;
+    uint256 private constant MAX_DELAY = 4 weeks;
```


## [G-05] State variable shadowed by function parameter
Variable shadowing should be avoided at all cost because the could make code review hard. So always ensure that local variables in the code do not shadow state variable.

### 1 Instance
- https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol#L42
```solidity
file: src/util/Factory.sol


36:    function newLiquidityPool(
37:        uint64 poolId,
38:        bytes16 trancheId,
39:        address currency,
40:        address trancheToken,
41:        address investmentManager,
42:        address[] calldata wards @audit variable shadows `wards` state variable.
43:    ) public auth returns (address) {
```

The `newLiquidityPool()` function of the `LiquidityPoolFactory` contract defined a `wards` parameter this `ward` parameter shadows the `ward` state variable inherited from the `Auth` contract. This is bad programming practice and it could make code review or debugging a bit difficult so its advisible to avoid this scenario. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/util/Factory.sol b/src/util/Factory.sol
index 7d4d5b3..977d12c 100644
--- a/src/util/Factory.sol
+++ b/src/util/Factory.sol
@@ -39,13 +39,13 @@ contract LiquidityPoolFactory is Auth {
         address currency,
         address trancheToken,
         address investmentManager,
-        address[] calldata wards
+        address[] calldata wardAddresses
     ) public auth returns (address) {
         LiquidityPool liquidityPool = new LiquidityPool(poolId, trancheId, currency, trancheToken, investmentManager);

         liquidityPool.rely(root);
-        for (uint256 i = 0; i < wards.length; i++) {
-            liquidityPool.rely(wards[i]);
+        for (uint256 i = 0; i < wardAddresses.length; i++) {
+            liquidityPool.rely(wardAddresses[i]);
         }
         liquidityPool.deny(address(this));
         return address(liquidityPool);
```


## CONCLUSIONS
As you embark on the journey of incorporating the recommended changes, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.
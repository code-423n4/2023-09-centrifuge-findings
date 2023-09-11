
**Low risk**
| count | Title |
|-------|-------|
|
| [L-01]   | Function does not follow checks Effects interactions (CEI) |
| [L-02]   | Lack of Validation for 'assets' Parameter in 'deposit' Function |
| [L-03]   | Missing Decimals Validation in addTranche Function|



| Total Low Risk Issues| 3 |
|-------|-------|



**Non-Critical**
| count | Title |
|-------|-------|
| 
| [NC-01]   | Recommendation for better variables Naming convention |
| [NC-02]   | Consider adding events, structs in there own file and import |
| [NC-03]   | use of floating numbers |
| [Nc-04]   | Simplify hasMember Function to improve readability|
| [NC-05]   | Its a good practice to add reentrancy guards where tokens are involved|



| Non-Critical Issues| 5 |
|-------|-------|

[L-01]: Function does not follow checks Effects interactions (CEI)
--------------------------------------------------------

```solidity
  function transferIn(address token, address source, address destination, uint256 amount) external auth {
        destinations[token][destination] += amount; 

        SafeTransferLib.safeTransferFrom(token, source, address(this), amount);
        emit TransferIn(token, source, destination, amount); 
    }
```
I would recommend that you emit event before you interact with other externals

[L-02]:  Missing Decimals Validation in addTranche Function
--------------------------------------------------------

```solidity
  function addTranche(
        uint64 poolId,
        bytes16 trancheId,
        string memory tokenName,
        string memory tokenSymbol,
        uint8 decimals
    ) public onlyGateway {
        Pool storage pool = pools[poolId];
        require(pool.createdAt != 0, "PoolManager/invalid-pool");
        Tranche storage tranche = pool.tranches[trancheId];
        require(tranche.createdAt == 0, "PoolManager/tranche-already-exists");

        tranche.poolId = poolId;
        tranche.trancheId = trancheId;
        tranche.decimals = decimals;
        tranche.tokenName = tokenName;
        tranche.tokenSymbol = tokenSymbol;
        tranche.createdAt = block.timestamp;

        emit TrancheAdded(poolId, trancheId);
    }
```

Impact:
In the addTranche function of the contract, there is a missing validation check for the decimals parameter. This omission could potentially lead to the addition of tranches with invalid or unexpected decimal values. The current implementation lacks a safeguard to ensure that the provided decimals value falls within a predefined valid range.

While the function is designed to add new tranches to an existing Centrifuge pool, it's crucial to validate the decimals value to prevent potential issues related to tokens with incorrect configurations. The absence of this validation could lead to tranches being added with inappropriate decimals values, which might result in unexpected behavior or vulnerabilities.

Recommendation:
To address this issue, it is recommended to implement a validation check for the decimals parameter in the addTranche function. This check should ensure that the provided decimals value falls within an acceptable range (e.g., between MIN_DECIMALS and MAX_DECIMALS). By doing so, you can enhance the security and reliability of the contract by preventing tranches with invalid decimal values from being added.

Validation Range: You mentioned adding a validation check to ensure that the provided decimals value falls within an acceptable range, such as between MIN_DECIMALS and MAX_DECIMALS. It's important to define these constants in your contract and use them in the validation to make the code more readable and maintainable.

Mitigation:
To mitigate this issue, add a validation check for the decimals parameter in the addTranche function to ensure it falls within an acceptable range.

This report highlights a potential improvement that can be made to enhance the security and reliability of the contract.


[L-03]:  Lack of Validation for 'assets' and 'shares' Parameter in 'deposit' Function 
------------------------------------------------------------------------

```solidity
 function deposit(uint256 assets, address receiver) public returns (uint256 shares) {
        shares = investmentManager.processDeposit(receiver, assets);
        emit Deposit(address(this), receiver, assets, shares);
    }

      function mint(uint256 shares, address receiver) public returns (uint256 assets) {
        // require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");
        assets = investmentManager.processMint(receiver, shares);
        emit Deposit(address(this), receiver, assets, shares);
    }
```
consider adding some checks to avoid sending 0 value assets and shares this would cause unexpected behaviours

**Non-critical issues**
-----------------------------


[NC-01]:  Recommendation for better variable naming 
------------------------------------------------------

Try to maintain naming consistency for your state variables thoroughout your code base so it will be easier to know what
you are dealing with , knowing out of the box you are dealing with a storage variable while using it will ensure you are aware of
gas optimization techniques

in your code base all storage, immutable and constant variables are using the same naming convention constants and immutable are all lowecase.
you might want to consider using this. to make it easier

for constants use

```solidity
uint256 public constant  CONSTANT_EXAMPLE = 1;

```

for storage variables use

```solidity

uint256 public  s_storageExample;

```

for immutable use

```solidity

uint256 private immutable i_immutable;

```

by doing this you will know out of the box what exactly you should do for simple optimizations like gas whenever you are dealing with storage variables.

[NC-02]: Consider adding events, structs in there own file and import
------------------------------------------------------------------------

in your codebase you are using  structs and events together with interfaces in contracts in one file this is making the file congested you can control this by making a separate file to add all events and structs and interfaces  this will make the code base more organised  so you just import them in the contract files that you need.

[NC-03]: Simplify hasMember Function to improve readability
-------------------------------
the function below has a lot of code that could be simplified to make the code better
```solidity
function hasMember(address user) public view returns (bool) {
        if (members[user] >= block.timestamp) {
            return true;
        }
        return false;
    }
```

you could consider changing this to 

```solidity
function hasMember(address user) public view returns (bool) {
    return members[user] >= block.timestamp;
}
```

[NC-04]: Its a good practice to add reentrancy guards where tokens are involved
-------------------------------

in your code you have multiple functions that deal with transfering tokens  i would advice that you make it a good practice that whenever you have to transfer tokens you add a noReentrancy guard to make the functions more secure,

below is an example of a function of transferring tokens that do not have a noReentrance guard on them

```solidity
 function transferFrom(address from, address to, uint256 value) public virtual returns (bool) {
        require(to != address(0) && to != address(this), "ERC20/invalid-address");
        uint256 balance = balanceOf[from];
        require(balance >= value, "ERC20/insufficient-balance");

        if (from != _msgSender()) {
            uint256 allowed = allowance[from][_msgSender()];
            if (allowed != type(uint256).max) {
                require(allowed >= value, "ERC20/insufficient-allowance");  
                unchecked {
                    allowance[from][_msgSender()] = allowed - value;
                }
            }
        }

        unchecked {
            balanceOf[from] = balance - value; 
            balanceOf[to] += value;
        }

        emit Transfer(from, to, value);

        return true;
    }
```
this code is found in ERC20.sol

[NC-05]: Use Floating Numbers
-------------------------------
in the following code you are using actual figures to verify the length

```solidity
  function _isValidSignature(address signer, bytes32 digest, bytes memory signature) internal view returns (bool) {
        if (signature.length == 65) {
            bytes32 r;
            bytes32 s;
            uint8 v;
            assembly {
                r := mload(add(signature, 0x20))
                s := mload(add(signature, 0x40))
                v := byte(0, mload(add(signature, 0x60)))
            }
            if (signer == ecrecover(digest, v, r, s)) {
                return true;
            }
        }

        (bool success, bytes memory result) =
            signer.staticcall(abi.encodeWithSelector(IERC1271.isValidSignature.selector, digest, signature));
        return (success && result.length == 32 && abi.decode(result, (bytes4)) == IERC1271.isValidSignature.selector);
    }
```

you could consider adding constant variables like say

 uint256 constant SIGNATURE_LENGTH = 65
uint256 constant RESULT_LENGTH = 32 

with this you can use 

return (success && result.length == RESULT_LENGTH && abi.decode(result, (bytes4))

this way when you login to the code later you will easily know what the length other than start wondering what 65 is







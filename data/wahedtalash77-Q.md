|     |
| --- |
| \[L-01\] Insufficient coverage |
| \[L-02\] Stop Using Solidity's transfer() Now |
| \[L-03\] ERC20 return values not checked |
| \[L-04\] `SafeTransfer` should be used in place of transfer |
| \[N-05\] Amounts should be checked for 0 before calling a transfer |
| \[N‑06\] Interfaces should be indicated with an `I` prefix in the contract name |
| \[N-07\] Interfaces should be moved to separate files |
| \[N-08\] Assembly Codes Specific – Should Have Comments |
| \[N-09\] Caching global variables is expensive than using the variable and there is no benefit of caching global variables |
| \[N‑10\] Large or complicated code bases should implement fuzzing tests |
| \[N-11\] Use named parameters for mapping type declarations |
| \[N-12\] Use of bytes.concat() instead of abi.encodePacked() |
| \[N-13\] NatSpec comments should be increased in contracts |
| \[N-14\] Function writing that does not comply with the Solidity Style Guide |
| \[N-15\] Use SMTChecker |
| \[N-16\] Function writing that does not comply with the Solidity Style Guide |
| \[S-17\] Generate perfect code headers every time |
| \[S-18\] You can explain the operation of critical functions in NatSpec with an infographic. |

# Low Risk

## \[L-01\] Insufficient coverage

Description
The test coverage rate of the project is ~80%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[L-02\] Stop Using Solidity's transfer() Now

**Proof Of Concept**
https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

```
169: emit Transfer(address(0), to, value);
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L169

```
192: emit Transfer(from, address(0), value);
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L192

## \[L-03\] ERC20 return values not checked

The ERC20.transfer() and ERC20.transferFrom() functions return a boolean value indicating success. This parameter needs to be checked for success. Some tokens do not revert if the transfer failed but return false instead.

```
File: src/InvestmentManager.sol

136: SafeTransferLib.safeTransferFrom(currency, user, address(escrow), _currencyAmount);

167:  lPool.transferFrom(user, address(escrow), _trancheTokenAmount);

313:   LiquidityPoolLike(liquidityPool).transferFrom(address(escrow), user, trancheTokenPayout),

475: lPool.transferFrom(address(escrow), user, trancheTokenAmount),
```

###Impact

Tokens that don’t actually perform the transfer and return false are still counted as a correct transfer and the tokens remain in the SingleNativeTokenExitV2 contract and could potentially be stolen by someone else.

### Recommended Mitigation Steps

We recommend using [OpenZeppelin’s](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.1/contracts/token/ERC20/utils/SafeERC20.sol#L74) `SafeERC20` versions with the `safeTransfer` and `safeTransferFrom` functions that handle the return value check as well as non-standard-compliant tokens.

## \[L-04\] `SafeTransfer` should be used in place of transfer

SafeTransfer should be used in place of Transfer for Solidity contracts to ensure robust security and error handling. Unlike the basic Transfer function, SafeTransfer incorporates safeguards against potential smart contract vulnerabilities, such as reentrancy attacks and unexpected token loss. By automatically validating the recipient's ability to receive tokens and reverting transactions in case of failures,

```
function transfer(uint128 token, address sender, bytes32 receiver, uint128 amount)
        public
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L192C2-L193C15

```
169: emit Transfer(address(0), to, value);
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L169

```
192: emit Transfer(from, address(0), value);
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L192

```
function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
        require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
        require(
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L36C2-L38C17

# Non-Critical

## \[N-01\] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```
function transfer(uint128 token, address sender, bytes32 receiver, uint128 amount)
        public
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L192C2-L193C15

```
169: emit Transfer(address(0), to, value);
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L169

```
192: emit Transfer(from, address(0), value);
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L192

```
function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
        require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
        require(
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L36C2-L38C17

## \[N‑02\] Interfaces should be indicated with an `I` prefix in the contract name

Examples:

```
File: src/LiquidityPool.sol

9: interface ERC20PermitLike {

File: src/InvestmentManager.sol

8: interface GatewayLike {


File: src/src/PoolManager.sol

11: interface GatewayLike {

File: src/Escrow.sol

7: interface ApproveLike {

...
```

### `I` prefix should be added to all contest.

## \[N-03\] Interfaces should be moved to separate files

> all contest

## \[N-04\] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of
Solidity, which can make the code more insecure and more error-prone.

```
if (!success) {
            assembly {
                let ptr := mload(0x40)
                let size := returndatasize()
                returndatacopy(ptr, 0, size)
                revert(ptr, size)
            }
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L339C8-L345C14

```
assembly {
            result := mload(add(source, 32))
        }
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Messages.sol#L882C7-L884C10

## \[N-05\] Caching global variables is expensive than using the variable and there is no benefit of caching global variables

```
118: address liquidityPool = msg.sender;
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L118

```
149: address liquidityPool = msg.sender;
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L149

```
File: src/InvestmentManager.sol

428:  address liquidityPool = msg.sender;

452:  address liquidityPool = msg.sender;

494 :  address liquidityPool = msg.sender;

520:  address liquidityPool = msg.sender;
```

## \[N‑06\] Large or complicated code bases should implement fuzzing tests

Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement <ins>fuzzing tests</ins>. Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

## \[N-07\] Use named parameters for mapping type declarations

Consider using named parameters in mappings (e.g. `mapping(address account => uint256 balance)`) to improve readability. This feature is present since Solidity 0.8.18.

```
File: src/PoolManager.sol


72:   mapping(address => address) liquidityPools; // currency -> liquidity pool address

91:   mapping(uint128 => address) public currencyIdToAddress;

92:   mapping(address => uint128) public currencyAddressToId;
```

```
26: mapping(address => bool) public liquidityPools;
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L26

```
mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    mapping(address => uint256) public nonces;
```

https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L25C2-L27C47

## \[N-08\] Use of bytes.concat() instead of abi.encodePacked()

```
242: permit(owner, spender, value, deadline, abi.encodePacked(r, s, v));
```

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

## \[N-09\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-10\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

\[N-04\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

## \[N-11\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

## \[N-12\] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

## \[S-01\] Generate perfect code headers every time

Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## \[S-02\] You can explain the operation of critical functions in NatSpec with an infographic.
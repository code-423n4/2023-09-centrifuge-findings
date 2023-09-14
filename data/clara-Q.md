# low report
# summary

| N     | Issue  | Instance |
|------|--------|----------|
|[L-01]|Usage of block.timestamp for comparisons in Root.executeScheduledRely(address)|1|
|[L-02]|Enhance Security by Checking the Return Value of transferFrom in InvestmentManager.sol|1|
|[L-03]|Potential Reentrancy in UserEscrow.transferOut Function|7|
|[L-04]|Correcting 'from' Parameter in InvestmentManager's transferFrom Function for ERC-20 TokensS|5|
|[L-05]|Missing Zero Address Validation in Constructor|1|
|[L-06]|Potential Reentrancy in Root.denyContract(address,address)|3|
|[L-07]|Replace Block.Timestamp Dependency with More Secure Timestamp Mechanism|4|
|[L-08]| Use of Low-Level Call in SafeTransferLib.safeApprove Function|3|





## [L-01] Usage of block.timestamp for comparisons in Root.executeScheduledRely(address)


In Solidity, the block.timestamp variable represents the current timestamp of the block being mined. It is commonly used for time-related operations in smart contracts, such as scheduling functions to execute at specific times.



Recommendation: Avoid relying on block.timestamp for critical logic. If time-based conditions are necessary, consider using block numbers instead. However, be aware that block time can vary and is not a precise measure of real-world time.


```solidity
    function executeScheduledRely(address target) public {
        require(schedule[target] != 0, "Root/target-not-scheduled");
        require(schedule[target] < block.timestamp, "Root/target-not-ready");

        wards[target] = 1;
        emit Rely(target);

        schedule[target] = 0;
    }
```
https://github.com/code-423n4/2023-09-centrifuge/blob/mainsrc/Root.sol#75-83

Fix code 
```solidity
//a new mapping to store block numbers can fix this issue
mapping(address => uint256) public scheduleBlock;

function scheduleRely(address target) external auth {
    scheduleBlock[target] = block.number + delay;
    emit RelyScheduled(target, scheduleBlock[target]);
}

function executeScheduledRely(address target) public {
    require(scheduleBlock[target] != 0, "Root/target-not-scheduled");
    require(scheduleBlock[target] < block.number, "Root/target-not-ready");

    wards[target] = 1;
    emit Rely(target);

    scheduleBlock[target] = 0;
}
```




## [L-02]Enhance Security by Checking the Return Value of transferFrom in InvestmentManager.sol

The return value of an external transfer/transferFrom call is not checked
To enhance security and mitigate risks in the InvestmentManager.sol contract, it is advisable to check the return value of the transferFrom function when performing token transfers. By verifying the return value, you can ensure that the transfer was successful and handle any potential errors or exceptions appropriately.

```solidity
167  lPool.transferFrom(user, address(escrow), _trancheTokenAmount);
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L167

Recommendation:
ensure that the transfer/transferFrom return value is checked , or use SafeERC20



## [L-03] Potential Reentrancy in UserEscrow.transferOut Function
In the UserEscrow contract, there is a potential vulnerability in the transferOut function that could allow for reentrancy attacks. This vulnerability arises from the sequence of external calls made within the function.
The transferOut function first calls the safeTransfer function from the SafeTransferLib library, which is an external contract. After that, it emits the TransferOut event.
```solidity
    function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
        require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
        require(
            /// @dev transferOut can only be initiated by the destination address or an authorized admin.
            ///      The check is just an additional protection to secure destination funds in case of compromized auth.
            ///      Since userEscrow is not able to decrease the allowance for the receiver,
            ///      a transfer is only possible in case receiver has received the full allowance from destination address.
            receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max),
            "UserEscrow/receiver-has-no-allowance"
        );
        destinations[token][destination] -= amount;

        SafeTransferLib.safeTransfer(token, receiver, amount);
        emit TransferOut(token, receiver, amount);
    }
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L36-L50


Recommendation:To mitigate the potential reentrancy vulnerability in the UserEscrow contract, it is advisable to follow the check-effects-interactions pattern. This pattern suggests organizing your code in a way that external calls are made at the end of the function, after all state changes and events have been handled.


replace code
```
function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
    require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
    require(receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max), "UserEscrow/receiver-has-no-allowance");

    destinations[token][destination] -= amount;

    emit TransferOut(token, receiver, amount);
    SafeTransferLib.safeTransfer(token, receiver, amount);
}

```
Impact: Implementing this change will enhance the security of the contract by preventing potential reentrancy attacks. 

another same issue in this function
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L29-L34




## [L-04]  Correcting 'from' Parameter in InvestmentManager's transferFrom Function for ERC-20 TokensS

In the provided code snippet, there is a potential issue where the transferFrom function is not using msg.sender as the from parameter.

```solidity
function _deposit(uint128 trancheTokenAmount, uint128 currencyAmount, address liquidityPool, address user)
    internal
{
    LiquidityPoolLike lPool = LiquidityPoolLike(liquidityPool);

    _decreaseDepositLimits(user, liquidityPool, currencyAmount, trancheTokenAmount); // decrease the possible deposit limits
    require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");
    require(
        lPool.transferFrom(address(escrow), user, trancheTokenAmount),
        "InvestmentManager/trancheTokens-transfer-failed"
    );

    emit DepositProcessed(liquidityPool, user, currencyAmount);
}

```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L467-L480



same issue in this link

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L136

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L291

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L167

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L312-L315



## [L-05] Missing Zero Address Validation in Constructor

The constructor of the Root contract accepts an address _escrow and assigns it to the state variable escrow. However, there is no check to ensure that the provided address is not the zero address. Assigning the zero address to escrow could lead to unexpected behavior and potential loss of funds, as the zero address is generally used to burn tokens and is not controlled by any user.


```solidity
34     constructor(address _escrow, uint256 _delay) {
        escrow = _escrow;
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L34-L35


Recommendation: 

This will cause the constructor to revert if the zero address is provided, preventing the contract from being deployed in an unsafe state.
```solidity
constructor(address _escrow, uint256 _delay) {
    require(_escrow != address(0), "Root/escrow-cannot-be-zero");
    escrow = _escrow;
    delay = _delay;

    wards[msg.sender] = 1;
    emit Rely(msg.sender);
}
```



## [L-06] Potential Reentrancy in Root.denyContract(address,address)


 The denyContract function makes an external call to AuthLike(target).deny(user) before emitting an event DenyContract(target,user). This could potentially lead to a reentrancy attack if the deny function in the AuthLike contract is not properly secured, causing the event to be emitted in an incorrect order.

```solidity
    function denyContract(address target, address user) public auth {
        AuthLike(target).deny(user);
        emit DenyContract(target, user);
    }
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L98-L101




### recommanded code
```solidity
function denyContract(address target, address user) public auth {
    emit DenyContract(target, user);
    AuthLike(target).deny(user);
}
```






## [L-07] Replace Block.Timestamp Dependency with More Secure Timestamp Mechanism

Dangerous usage of block.timestamp. block.timestamp can be manipulated by miners.

```solidity
46    require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");

50   if (members[user] >= block.timestamp) {

58   require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");    
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L46
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L50
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L58


```solidity
217   require(block.timestamp <= deadline, "ERC20/permit-expired");
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L217

Recommendation:
Avoid relying on block.timestamp.




## [L-08] Use of Low-Level Call in SafeTransferLib.safeApprove Function

 The safeApprove function in the SafeTransferLib contract uses a low-level call to a custom address. Low-level calls in Solidity can introduce potential security risks, including reentrancy attacks and unexpected failures due to out-of-gas errors. The use of low-level calls also reduces code readability and maintainability, making it more difficult to identify and fix potential issues.

```solidity
    function safeApprove(address token, address to, uint256 value) internal {
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(IERC20.approve.selector, to, value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");
    }
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/SafeTransferLib.sol#L36-L39



The recommendation is to avoid using low-level calls whenever possible. Low-level calls, such as call() or delegatecall(), can be complex to handle correctly and increase the risk of vulnerabilities, including reentrancy attacks and unexpected behavior. By relying on Solidity's high-level functions, such as transfer() or send(), you can leverage their built-in safety mechanisms and reduce the likelihood of encountering such issues.

#### same issue 

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/SafeTransferLib.sol#L26-L29

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/SafeTransferLib.sol#L15-L19

## [L-01] Missing Zero Address Validation in Constructor

```solidity
34     constructor(address _escrow, uint256 _delay) {
        escrow = _escrow;
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L34-L35

Description: The constructor of the Root contract accepts an address _escrow and assigns it to the state variable escrow. However, there is no check to ensure that the provided address is not the zero address. Assigning the zero address to escrow could lead to unexpected behavior and potential loss of funds, as the zero address is generally used to burn tokens and is not controlled by any user.

Recommendation: 
```solidity
constructor(address _escrow, uint256 _delay) {
    require(_escrow != address(0), "Root/escrow-cannot-be-zero");
    escrow = _escrow;
    delay = _delay;

    wards[msg.sender] = 1;
    emit Rely(msg.sender);
}
```
This will cause the constructor to revert if the zero address is provided, preventing the contract from being deployed in an unsafe state.


## [L-02] Potential Reentrancy in Root.denyContract(address,address)

```solidity
    function denyContract(address target, address user) public auth {
        AuthLike(target).deny(user);
        emit DenyContract(target, user);
    }
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L98-L101

`Description:` The denyContract function makes an external call to AuthLike(target).deny(user) before emitting an event DenyContract(target,user). This could potentially lead to a reentrancy attack if the deny function in the AuthLike contract is not properly secured, causing the event to be emitted in an incorrect order.

`Recommendation:` Apply the check-effects-interactions pattern to prevent potential reentrancy. Emit the event before making the external call.

`Fix code:`
```solidity
function denyContract(address target, address user) public auth {
    emit DenyContract(target, user);
    AuthLike(target).deny(user);
}
```

same issue also here 
```solidity
    function relyContract(address target, address user) public auth {
        AuthLike(target).rely(user);
        emit RelyContract(target, user);
    }
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L98-L101


## [L-03] Usage of block.timestamp for comparisons in Root.executeScheduledRely(address)

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

Description: The function executeScheduledRely uses block.timestamp for comparisons. This could potentially be manipulated by miners, leading to unexpected behavior.

Recommendation: Avoid relying on block.timestamp for critical logic. If time-based conditions are necessary, consider using block numbers instead. However, be aware that block time can vary and is not a precise measure of real-world time.

Fix code 
```solidity
// Define a new mapping to store block numbers
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

## [L-04] Replace Block.Timestamp Dependency with More Secure Timestamp Mechanism
Dangerous usage of block.timestamp. block.timestamp can be manipulated by miners.

```solidity
46    require((members[user] >= block.timestamp), "RestrictionManager/destination-not-a-member");

50   if (members[user] >= block.timestamp) {

58   require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");    
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol#L46

```solidity
217   require(block.timestamp <= deadline, "ERC20/permit-expired");
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L217

Recommendation:
Avoid relying on block.timestamp.




## [L-05] Use of Low-Level Call in SafeTransferLib.safeApprove Function
Description: The safeApprove function in the SafeTransferLib contract uses a low-level call to a custom address. Low-level calls in Solidity can introduce potential security risks, including reentrancy attacks and unexpected failures due to out-of-gas errors. The use of low-level calls also reduces code readability and maintainability, making it more difficult to identify and fix potential issues.

```solidity
    function safeApprove(address token, address to, uint256 value) internal {
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(IERC20.approve.selector, to, value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");
    }
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/SafeTransferLib.sol#L36-L39



Recommendation: To mitigate these risks, it is recommended to avoid low-level calls whenever possible and use Solidity's high-level functions instead. If a low-level call is necessary, it should be wrapped in a require statement to ensure that the call was successful.

#### same issue in these functions

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/SafeTransferLib.sol#L26-L29

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/SafeTransferLib.sol#L15-L19



## [L-06] Potential Reentrancy in UserEscrow.transferOut Function
Description: The transferOut function in the UserEscrow contract has a potential reentrancy vulnerability. The function makes an external call to SafeTransferLib.safeTransfer before emitting an event TransferOut. If the safeTransfer function calls back into the UserEscrow contract, it could lead to out-of-order events or other unexpected behavior.

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


Recommendation: To mitigate this risk, it is recommended to apply the check-effects-interactions pattern. This pattern suggests that you should make all external calls at the end of the function, after all state changes and events.

Fix code
```
function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
    require(destinations[token][destination] >= amount, "UserEscrow/transfer-failed");
    require(receiver == destination || (ERC20Like(token).allowance(destination, receiver) == type(uint256).max), "UserEscrow/receiver-has-no-allowance");

    destinations[token][destination] -= amount;

    emit TransferOut(token, receiver, amount);
    SafeTransferLib.safeTransfer(token, receiver, amount);
}

```
Impact: Implementing this change will enhance the security of the contract by preventing potential reentrancy attacks. It will ensure that all state changes and events occur before any external calls, reducing the potential attack surface.

another same issue in this function
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol#L29-L34


## [L-07]  Correcting 'from' Parameter in InvestmentManager's transferFrom Function for ERC-20 TokensS

Issue Description:
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

Recommendation:
The recommendation is to use msg.sender as the from parameter in the transferFrom function to ensure that the tokens are transferred from the sender's address.

Fixed Code:
Here is the modified code with the recommended change:

```solidity
function _deposit(uint128 trancheTokenAmount, uint128 currencyAmount, address liquidityPool, address user)
    internal
{
    LiquidityPoolLike lPool = LiquidityPoolLike(liquidityPool);

    _decreaseDepositLimits(user, liquidityPool, currencyAmount, trancheTokenAmount); // decrease the possible deposit limits
    require(lPool.checkTransferRestriction(msg.sender, user, 0), "InvestmentManager/trancheTokens-not-a-member");
    require(
        lPool.transferFrom(msg.sender, user, trancheTokenAmount), // Use msg.sender as 'from'
        "InvestmentManager/trancheTokens-transfer-failed"
    );

    emit DepositProcessed(liquidityPool, user, currencyAmount);
}

```



same issue in this link

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L136

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L291

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L167

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L312-L315

## [L-08]Enhance Security by Checking the Return Value of transferFrom in InvestmentManager.sol

```solidity
167  lPool.transferFrom(user, address(escrow), _trancheTokenAmount);
```
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol#L167
Description:
The return value of an external transfer/transferFrom call is not checked

Recommendation:
Use SafeERC20, or ensure that the transfer/transferFrom return value is checked.


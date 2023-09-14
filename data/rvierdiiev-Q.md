## QA-01. Messages._stringToBytes32 will truncate data for messages that are longer than 32 bytes

### Details
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol#L876-L885
```solidity
    function _stringToBytes32(string memory source) internal pure returns (bytes32 result) {
        bytes memory tempEmptyStringTest = bytes(source);
        if (tempEmptyStringTest.length == 0) {
            return 0x0;
        }


        assembly {
            result := mload(add(source, 32))
        }
    }
```
In order to convert string to bytes32, function just returns 32 bytes, But in case if string is bigger than 32 bytes, then it will truncate string and provide wrong data.

## Recommended Mitigation Steps
Check that string is not bigger than 32 bytes.

## QA-02. Users can call PoolManager.transferTrancheTokensToCentrifuge a lot of times on cheap chains in order to flood centrifuge chain with messages.

### Details
All messages that are relied to the centrifuge chain from Gateway are then executed by protocol's bot. This bot will pay gas fees for execution.

Users on cheap chains can call `PoolManager.transferTrancheTokensToCentrifuge` with 0 amount just to pass message on centrifuge chain. As result they can spam with big amount of messages, which bots should execute or skip.

## Recommended Mitigation Steps
Do not allow bridge 0 tokens.

## QA-03. increaseAllowance/decreaseAllowance may not work with some wallets.

### Details
Recently, open zeppelin has remove `increaseAllowance/decreaseAllowance` function form ERC20 contract.
This is because they are not described in the standard and [as described in the disscussion](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4583#issuecomment-1710297819) such function may not work well with some wallets that are trying to limit user's activity(spending/approving).

But ERC20 of centrifuge protocol, [still has such functions](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L139-L159).
## Recommended Mitigation Steps
Think, if it's worth removing them as well.
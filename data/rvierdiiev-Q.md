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
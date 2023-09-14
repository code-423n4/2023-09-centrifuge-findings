## The Messages._stringToBytes128() and _stringToBytes32 does not verify whether the `string source` argument exceeds bytes in length.

To address this concern, a require statement is used to validate that the input string's length `(bytes(source).length)` does not exceed 128 bytes. If the string's length exceeds 128 bytes, the transaction is reverted, accompanied by the error message "String exceeds 128 bytes." This approach ensures the safe conversion of the input string without data loss, as the function only continues if the length requirement is met, and it terminates the transaction if the requirement is not met.

```diff
function _stringToBytes128(
        string memory source
    ) internal pure returns (bytes memory) {
+       require(bytes(source).length <= 128, "String exceeds 128 bytes");
        bytes memory temp = bytes(source);
        bytes memory result = new bytes(128);

        // for loop

        return result;
    }

function _stringToBytes32(
        string memory source
    ) internal pure returns (bytes32 result) {
+       require(bytes(source).length <= 32, "String exceeds 128 bytes");
        bytes memory tempEmptyStringTest = bytes(source);
        if (tempEmptyStringTest.length == 0) {
            return 0x0;
        }

        assembly {
            result := mload(add(source, 32))
        }
    }
```

## Useless `liquidityPool` argument in `InvestmentManager._toPriceDecimals` function
Consider removing the useless `liquidityPool` argument in the `InvestmentManager._toPriceDecimals` function.
```diff
function _toPriceDecimals(
        uint128 _value,
        uint8 decimals,
-        address liquidityPool
    ) internal view returns (uint256 value) {
        if (PRICE_DECIMALS == decimals) return uint256(_value);
        value = uint256(_value) * 10 ** (PRICE_DECIMALS - decimals);
    }
```
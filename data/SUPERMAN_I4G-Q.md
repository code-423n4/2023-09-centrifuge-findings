**Add to Analysis Report**

**Code Line Reviewed:**

Line: 167
lPool.transferFrom(user, address(escrow), _trancheTokenAmount);


**Summary:**

The analysis pertains to the specific line of code within the `requestRedeem` function where Slither has issued a warning for "unchecked-transfer." Below is an assessment of this line:

**Code Line:**

lPool.transferFrom(user, address(escrow), _trancheTokenAmount);


- **Issue:** Slither has issued a high severity warning for "unchecked-transfer," indicating that the return value of an external `transferFrom` call is not checked.

- **Explanation:** The code line invokes the `transferFrom` function on the `lPool` contract to transfer `_trancheTokenAmount` tokens from the `user` to the `escrow` address. However, it does not check or handle the return value of this operation.

- **Recommendation:** It's essential to handle the return value of external token transfer functions, such as `transferFrom`. These functions can return a boolean value indicating whether the transfer was successful. Ignoring this return value may result in unhandled errors or unexpected behaviour. Consider implementing error-handling logic to respond appropriately in case the transfer fails. For example, you can use `require` statements to ensure that the transfer succeeds or revert the transaction with a meaningful error message if it fails.

**Conclusion:**

The code line `lPool.transferFrom(user, address(escrow), _trancheTokenAmount);` within the `requestRedeem` function invokes an external token transfer operation without checking or handling the return value. This can lead to unhandled errors or unexpected behaviour if the transfer fails. To ensure robust error handling, consider implementing checks and error handling logic to respond appropriately to the outcome of the transfer operation.

**Tool used:**

 Slither

**Severity:**

 High

**Reference:**

https://github.com/crytic/slither/wiki/Detector-Documentation#unchecked-transfer
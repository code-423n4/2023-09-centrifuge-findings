**Code Analysis Report**

**Lines Reviewed:**

Line 124 - poolManager.isAllowedAsPoolCurrency(lPool.poolId(), currency); 
Line 154 - poolManager.isAllowedAsPoolCurrency(lPool.poolId(), lPool.asset());

**Summary:**

The analysis pertains to two lines of code within their respective functions where Slither has identified that the return value of the function `poolManager.isAllowedAsPoolCurrency` is ignored. Below is an assessment of each line:

**Line 124:**

poolManager.isAllowedAsPoolCurrency(lPool.poolId(), currency);


- **Issue:** Slither has flagged this line as "ignores return value."

- **Explanation:** The function `poolManager.isAllowedAsPoolCurrency` is called with arguments `lPool.poolId()` and `currency`, but its return value is not captured or utilized in the code.

- **Recommendation:** Consider reviewing the code to determine whether the return value of this function call should be used for program logic or error handling. If the return value is essential, capture it in a variable or use it appropriately.

**Line 154:**

poolManager.isAllowedAsPoolCurrency(lPool.poolId(), lPool.asset());

- **Issue:** Slither has flagged this line as "ignores return value."

- **Explanation:** Similar to the previous line, the function `poolManager.isAllowedAsPoolCurrency` is called with arguments `lPool.poolId()` and `lPool.asset()`, but its return value is not captured or utilized in the code.

- **Recommendation:** Examine the code to determine if the return value of this function call is significant for logic behaviour or requirements. If it holds importance, consider capturing the return value or utilizing it as necessary.

**Conclusion:**

In both lines of code, the function `poolManager.isAllowedAsPoolCurrency` is called without capturing or using its return value. Slither has flagged these lines due to the ignored return value. It is advisable to review the code to determine if these return values should be considered for the contract logic or error handling, and if so, take appropriate actions to use them effectively.

**Severity**
Medium

**Reference**
https://github.com/crytic/slither/wiki/Detector-Documentation#unused-return

### Time spent:
16 hours
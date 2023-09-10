
# delay must be checked in Root constructor
`_delay` value is not checked by the constructor, and so can exceed `MAX_DELAY` value.

Add the following statement to constructor:

```solidity
require(_delay <= MAX_DELAY, "Root/delay-too-long");
```



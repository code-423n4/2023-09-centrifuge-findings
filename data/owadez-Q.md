 executeScheduledRely() in root.sol will not work correctly because block.timestamp is used without delay .
```
   function executeScheduledRely(address target) public {
        require(schedule[target] != 0, "Root/target-not-scheduled");
        require(schedule[target] < block.timestamp, "Root/target-not-ready"); 

        wards[target] = 1;
        emit Rely(target);

        schedule[target] = 0;
    }

```
The following should be used
```
 require(schedule[target] < block.timestamp + delay , "Root/target-not-ready");
```

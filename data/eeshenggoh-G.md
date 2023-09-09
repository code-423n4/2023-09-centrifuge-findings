# Use nested if and avoid multiple check combinations

Using nested if, is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```
        require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-from-failed");
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/util/SafeTransferLib.sol#L18C27-L18C27

```
        require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-transfer-failed");
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/util/SafeTransferLib.sol#L28

```
        require(success && (data.length == 0 || abi.decode(data, (bool))), "SafeTransferLib/safe-approve-failed");
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/util/SafeTransferLib.sol#L38C9-L38C9

Simple Test Case:
```
contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.checkAge(19);
        c1.checkAgeOptimized(19);
    }
}
contract Contract0 {
    function checkAge(uint8 _age) public returns(string memory){
        if(_age>18 && _age<22){
            return "Eligible";
        }
    }
}
contract Contract1 {
    function checkAgeOptimized(uint8 _age) public returns(string memory){
        if(_age>18){
            if(_age<22){
                return "Eligible";
            }
        }
    }
}
```
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

# Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```
        for (uint256 i = 0; data. Length+) {
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/gateway/Messages.sol#L848C44-L848C44

```
        for (uint8 j = 0; j < i; j++) {
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/gateway/Messages.sol#L869

```
               for (i = 0; i < 32 && _bytes32[i] != 0; i++) {
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/gateway/Messages.sol#L893

```
        for (uint256 i = 0; i < userLength; i++) {
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/token/RestrictionManager.sol#L64

```
        for (uint256 i = 0; i < wards. Length; i++) {
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/util/Factory.sol#L47
```
        for (uint256 i = 0; i < trancheTokenWards.length; i++) {
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/util/Factory.sol#L103

```
        for (uint256 i = 0; i < restrictionManagerWards.length; i++) {
```
https://github.com/code-423n4/2023-09-centrifuge/blob/22e3976b9b2f0977924d07d201dd0a6c824eb43f/src/util/Factory.sol#L117
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
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}
```
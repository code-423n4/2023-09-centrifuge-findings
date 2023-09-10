**src/Escrow.sol**
- L7 - The ApproveLike interface is created, which is never used throughout the contract, therefore it would be best to eliminate it so as not to take up unnecessary space.


**src/Root.sol**
- L35 - Escrow variable is immutable and the constructor does not validate that it is an address other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.


**src/admins/PauseAdmin.sol**
- L22 - Root variable is immutable and the constructor does not validate that it is an address other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.


**src/admins/DelayedAdmin.sol**
- L19 - Root variable is immutable and the constructor does not validate that it is an address other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.


**src/token/Tranche.sol**
- L7 - The TrancheTokenLike interface is created, but it is never used on the contrary, therefore it should be eliminated.


**src/token/RestrictionManager.sol**
- L6 - The MemberlistLike interface is created, but it is never used on the contrary, therefore it should be eliminated.


**src/util/Factory.sol**
- L30/75 - Root variable is immutable and the constructor does not validate that it is an address other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.

- L94 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**src/gateway/routers/axelar/Router.sol**
- L37 - Root variable is immutable and the constructor does not validate that it is an address other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.


**src/LiquidityPool.sol**
- L86/87/88/89 - The variables: poolId, trancheId, asset, and share are immutable and the constructor does not validate that they are addresses other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.


**src/InvestmentManager.sol**
- L89/90 - The variables: escrow and userEscrow are immutable and the constructor does not validate that they are addresses other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.


**src/gateway/Gateway.sol**
- L99 - Root variable is immutable and the constructor does not validate that it is an address other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.

- L285 - The handle function has 81 lines of code and many conditionals, this reduces the understanding of the code. It would be beneficial for the users' understanding of this contract to have auxiliary functions with names that represent them.


**src/token/ERC20.sol**
- L225 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**src/PoolManager.sol**
- L105/106/107 - The variables: escrow, liquidityPoolFactory and trancheTokenFactory are immutable and the constructor does not validate that they are addresses other than 0x. This should be validated so as not to enter incorrect states and render the contract unusable.


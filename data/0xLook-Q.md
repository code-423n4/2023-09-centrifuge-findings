# The comment is wrong

LiquidityPool.sol Line [96](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L96)
```
96:    /// @dev Either msg.sender is the owner or a ward on the contract @audit comment is wrong
97:    modifier withApproval(address owner) {
98:        require(msg.sender == owner, "LiquidityPool/no-approval");
99:        _;
100:    }
```
This modifier should only check that the owner is the msg.sender. 

# Remove commented out code
LiquidityPool.sol Line [149](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L149)
```
148:    function mint(uint256 shares, address receiver) public returns (uint256 assets) {
149:        // require(receiver == msg.sender, "LiquidityPool/not-authorized-to-mint");
150:        assets = investmentManager.processMint(receiver, shares);
151:        emit Deposit(address(this), receiver, assets, shares);
152:    }
```

# No same value input control 
[Auth.sol#L14-L17](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Auth.sol#L14-L17)
```
14:    function rely(address user) external auth {
15:        wards[user] = 1;
16:        emit Rely(user);
17:    }
```
[Auth.sol#L20-L23](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Auth.sol#L20-L23)
```
20:    function deny(address user) external auth {
21:        wards[user] = 0;
22:        emit Deny(user);
23:    }
```
[PauseAdmin#L34-L37](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L34-L37)
```
34:    function addPauser(address user) external auth {
35:        pausers[user] = 1;
36:        emit AddPauser(user);
37:    }
```
[PauseAdmin#L39-L42](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L39-L42)
```
39:    function removePauser(address user) external auth {
40:        pausers[user] = 0;
41:        emit RemovePauser(user);
42:    }
```
# According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
[InvestmentManager#L81](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/InvestmentManager.sol#L81)
```
81:     mapping(address => mapping(address => LPValues)) public orderbook;
```
[PoolManager#L56-L57](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L56-L57)
```
56:    mapping(bytes16 => Tranche) tranches;
57:    mapping(address => bool) allowedCurrencies;
```
[PoolManager#L72](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L72)
```
72:    mapping(address => address) liquidityPools; // currency -> liquidity pool address
```
[PoolManager#L88](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L88)
```
88:     mapping(uint64 => Pool) public pools;
```
[PoolManager#L91-L92](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/PoolManager.sol#L91-L92)
```
91:    mapping(uint128 => address) public currencyIdToAddress;
92:    mapping(address => uint128) public currencyAddressToId;
```
[Root#L21](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L21)
```
21:    mapping(address => uint256) public schedule;
```
[UseEscrow#L17](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/UserEscrow.sol#L17)
```
17:    mapping(address => mapping(address => uint256)) destinations;
```
[PauseAdmin#L13](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/admins/PauseAdmin.sol#L13)
```
13:    mapping(address => uint256) public pausers;
```
[Gateway#L89](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/gateway/Gateway.sol#L89)
```
89:    mapping(address => bool) public incomingRouters;
```
[ERC20#L17](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L17)
```
17:    mapping(address => uint256) public wards;
```
[ERC20#L25-L26-L27](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L25-L27)
```
25:    mapping(address => uint256) public balanceOf;
26:    mapping(address => mapping(address => uint256)) public allowance;
27:    mapping(address => uint256) public nonces;
```
[RestrictionManager#L20](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/RestrictionManager.sol#L20)
```
20:    mapping(address => uint256) public members;
```
[Tranche#L26](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/Tranche.sol#L26)
```
26:    mapping(address => bool) public liquidityPools;
```
[Auth#L8](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Auth.sol#L8)
```
8:    mapping(address => uint256) public wards;
```
# Use SMTChecker
the highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.
https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

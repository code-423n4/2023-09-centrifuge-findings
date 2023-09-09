## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Auth.sol

```solidity
// place this modifier before the constructor
26:    modifier auth() {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/routers/axelar/Router.sol

```solidity
// place these modifiers before the constructor
43:    modifier onlyCentrifugeChainOrigin(string calldata sourceChain, string calldata sourceAddress) {
56:    modifier onlyGateway() {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol

```solidity
// place these modifiers before the constructor
109:    modifier onlyInvestmentManager() {
114:    modifier onlyPoolManager() {
119:    modifier onlyIncomingRouter() {
124:    modifier pauseable() {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol

```solidity
// place this modifier before the constructor
51:    modifier auth() {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol

```solidity
// place this modifier before the constructor
35:    modifier restricted(address from, address to, uint256 value) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/PauseAdmin.sol

```solidity
// place this modifier before the constructor
28:    modifier canPause() {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol

```solidity
// place this modifier before the constructor
114:    modifier onlyGateway() {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol

```solidity
// place this modifier before the constructor
97:    modifier onlyGateway() {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol

```solidity
// place this modifier before the constructor
97:    modifier withApproval(address owner) {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol

```solidity
// place these external function first
285:    function handle(bytes calldata message) external onlyIncomingRouter pauseable {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol

```solidity
// place this private for last
67:    function _calculateDomainSeparator(uint256 chainId) private view returns (bytes32) {

// place these external on top with the other externals
131:    function approve(address spender, uint256 value) external returns (bool) {
139:    function increaseAllowance(address spender, uint256 addedValue) external returns (bool) {
148:    function decreaseAllowance(address spender, uint256 subtractedValue) external returns (bool) {
172:    function burn(address from, uint256 value) external auth {
239:    function permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)

// place this internal before all private functions
196:    function _isValidSignature(address signer, bytes32 digest, bytes memory signature) internal view returns (bool) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol

```solidity
// place these externals before public functions.
155:    function maxMint(address receiver) external view returns (uint256 maxShares) {
160:    function previewMint(uint256 shares) external view returns (uint256 assets) {
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol

```solidity
9:  interface RootLike {
10:    function escrow() external view returns (address);
13:  interface LiquidityPoolFactoryLike {
14:    function newLiquidityPool(
29:    constructor(address _root) {
36:    function newLiquidityPool(
55:  interface TrancheTokenFactoryLike {
56:    function newTrancheToken(
74:    constructor(address _root) {
81:    function newTrancheToken(
111:    function _newRestrictionManager(address[] calldata restrictionManagerWards) internal returns (address memberList) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Context.sol

```solidity
13:    function _msgSender() internal view virtual returns (address) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Auth.sol

```solidity
10:    event Rely(address indexed user);
11:    event Deny(address indexed user);
14:    function rely(address user) external auth {
20:    function deny(address user) external auth {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/routers/axelar/Router.sol

```solidity
6:  interface AxelarGatewayLike {
7:    function callContract(string calldata destinationChain, string calldata contractAddress, bytes calldata payload)
10:    function validateContractCall(
18:  interface GatewayLike {
19:    function handle(bytes memory message) external;
34:    event File(bytes32 indexed what, address addr);
36:    constructor(address axelarGateway_) {
43:    modifier onlyCentrifugeChainOrigin(string calldata sourceChain, string calldata sourceAddress) {
56:    modifier onlyGateway() {
62:    function file(bytes32 what, address data) external auth {
73:    function execute(
89:    function send(bytes calldata message) public onlyGateway {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol

```solidity
63:    enum Domain {
68:    function messageType(bytes memory _msg) internal pure returns (Call _call) {
83:    function isAddCurrency(bytes memory _msg) internal pure returns (bool) {
87:    function parseAddCurrency(bytes memory _msg) internal pure returns (uint128 currency, address currencyAddress) {
102:    function isAddPool(bytes memory _msg) internal pure returns (bool) {
106:    function parseAddPool(bytes memory _msg) internal pure returns (uint64 poolId) {
121:    function isAllowPoolCurrency(bytes memory _msg) internal pure returns (bool) {
125:    function parseAllowPoolCurrency(bytes memory _msg) internal pure returns (uint64 poolId, uint128 currency) {
163:    function isAddTranche(bytes memory _msg) internal pure returns (bool) {
167:    function parseAddTranche(bytes memory _msg)
205:    function formatUpdateMember(uint64 poolId, bytes16 trancheId, bytes32 member, uint64 validUntil)
213:    function isUpdateMember(bytes memory _msg) internal pure returns (bool) {
217:    function parseUpdateMember(bytes memory _msg)
245:    function isUpdateTrancheTokenPrice(bytes memory _msg) internal pure returns (bool) {
249:    function parseUpdateTrancheTokenPrice(bytes memory _msg)
277:    function isTransfer(bytes memory _msg) internal pure returns (bool) {
281:    function parseTransfer(bytes memory _msg)
294:    function parseIncomingTransfer(bytes memory _msg)
345:    function isTransferTrancheTokens(bytes memory _msg) internal pure returns (bool) {
351:    function parseTransferTrancheTokens20(bytes memory _msg)
384:    function isIncreaseInvestOrder(bytes memory _msg) internal pure returns (bool) {
388:    function parseIncreaseInvestOrder(bytes memory _msg)
420:    function isDecreaseInvestOrder(bytes memory _msg) internal pure returns (bool) {
424:    function parseDecreaseInvestOrder(bytes memory _msg)
452:    function isIncreaseRedeemOrder(bytes memory _msg) internal pure returns (bool) {
456:    function parseIncreaseRedeemOrder(bytes memory _msg)
484:    function isDecreaseRedeemOrder(bytes memory _msg) internal pure returns (bool) {
488:    function parseDecreaseRedeemOrder(bytes memory _msg)
512:    function isCollectInvest(bytes memory _msg) internal pure returns (bool) {
516:    function parseCollectInvest(bytes memory _msg)
543:    function isCollectRedeem(bytes memory _msg) internal pure returns (bool) {
547:    function parseCollectRedeem(bytes memory _msg)
558:    function formatExecutedDecreaseInvestOrder(
570:    function isExecutedDecreaseInvestOrder(bytes memory _msg) internal pure returns (bool) {
574:    function parseExecutedDecreaseInvestOrder(bytes memory _msg)
586:    function formatExecutedDecreaseRedeemOrder(
598:    function isExecutedDecreaseRedeemOrder(bytes memory _msg) internal pure returns (bool) {
602:    function parseExecutedDecreaseRedeemOrder(bytes memory _msg)
614:    function formatExecutedCollectInvest(
633:    function isExecutedCollectInvest(bytes memory _msg) internal pure returns (bool) {
637:    function parseExecutedCollectInvest(bytes memory _msg)
657:    function formatExecutedCollectRedeem(
676:    function isExecutedCollectRedeem(bytes memory _msg) internal pure returns (bool) {
680:    function parseExecutedCollectRedeem(bytes memory _msg)
700:    function parseExecutedCollectRedeem(bytes memory _msg)
704:    function isScheduleUpgrade(bytes memory _msg) internal pure returns (bool) {
708:    function parseScheduleUpgrade(bytes memory _msg) internal pure returns (address _contract) {
712:    function formatCancelUpgrade(address _contract) internal pure returns (bytes memory) {
716:    function isCancelUpgrade(bytes memory _msg) internal pure returns (bool) {
720:    function parseCancelUpgrade(bytes memory _msg) internal pure returns (address _contract) {
751:    function isUpdateTrancheTokenMetadata(bytes memory _msg) internal pure returns (bool) {
755:    function parseUpdateTrancheTokenMetadata(bytes memory _msg)
766:    function formatCancelInvestOrder(uint64 poolId, bytes16 trancheId, bytes32 investor, uint128 currency)
774:    function parseCancelInvestOrder(bytes memory _msg)
785:    function formatCancelRedeemOrder(uint64 poolId, bytes16 trancheId, bytes32 investor, uint128 currency)
793:    function parseCancelRedeemOrder(bytes memory _msg)
820:    function isUpdateTrancheInvestmentLimit(bytes memory _msg) internal pure returns (bool) {
824:    function parseUpdateTrancheInvestmentLimit(bytes memory _msg)
836:    function formatDomain(Domain domain) public pure returns (bytes9) {
840:    function formatDomain(Domain domain, uint64 chainId) public pure returns (bytes9) {
844:    function _stringToBytes128(string memory source) internal pure returns (bytes memory) {
859:    function _bytes128ToString(bytes memory _bytes128) internal pure returns (string memory) {
876:    function _stringToBytes32(string memory source) internal pure returns (bytes32 result) {
887:    function _bytes32ToString(bytes32 _bytes32) internal pure returns (string memory) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Gateway.sol

```solidity
7:interface InvestmentManagerLike {
8:    function updateTrancheTokenPrice(uint64 poolId, bytes16 trancheId, uint128 currencyId, uint128 price) external;
9:    function handleExecutedDecreaseInvestOrder(
16:    function handleExecutedDecreaseRedeemOrder(
23:    function handleExecutedCollectInvest(
31:    function handleExecutedCollectRedeem(
41:interface PoolManagerLike {
42:    function addPool(uint64 poolId) external;
43:    function allowPoolCurrency(uint64 poolId, uint128 currency) external;
44:    function addTranche(
51:    function updateMember(uint64 poolId, bytes16 trancheId, address user, uint64 validUntil) external;
52:    function updateTrancheTokenMetadata(
58:    function addCurrency(uint128 currency, address currencyAddress) external;
59:    function handleTransfer(uint128 currency, address recipient, uint128 amount) external;
60:    function handleTransferTrancheTokens(uint64 poolId, bytes16 trancheId, address destinationAddress, uint128 amount)
62:interface RouterLike {
65:    function send(bytes memory message) external;
68:interface AuthLike {
69:    function rely(address usr) external;
72:interface RootLike {
73:    function paused() external returns (bool);
74:    function scheduleRely(address target) external;
75:    function cancelRely(address target) external;
93:    event AddIncomingRouter(address indexed router);
94:    event RemoveIncomingRouter(address indexed router);
95:    event UpdateOutgoingRouter(address indexed router);
96:    event File(bytes32 indexed what, address data);
98:    constructor(address root_, address investmentManager_, address poolManager_, address router_) {
109:    modifier onlyInvestmentManager() {
114:    modifier onlyPoolManager() {
119:    modifier onlyIncomingRouter() {
124:    modifier pauseable() {
130:    function file(bytes32 what, address data) public auth {
137:    function addIncomingRouter(address router) public auth {
142:    function removeIncomingRouter(address router) public auth {
147:    function updateOutgoingRouter(address router) public auth {
153:    function transferTrancheTokensToCentrifuge(
172:    function transferTrancheTokensToEVM(
192:    function transfer(uint128 token, address sender, bytes32 receiver, uint128 amount)
200:    function increaseInvestOrder(
212:    function decreaseInvestOrder(
224:    function increaseRedeemOrder(
238:    function decreaseRedeemOrder(
252:    function collectInvest(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
260:    function collectRedeem(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
268:    function cancelInvestOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
276:    function cancelRedeemOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
285:    function handle(bytes calldata message) external onlyIncomingRouter pauseable {
369:    function _addressToBytes32(address x) internal pure returns (bytes32) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/RestrictionManager.sol

```solidity
6:  interface MemberlistLike {
7:    function updateMember(address user, uint256 validUntil) external;
8:    function members(address user) external view returns (uint256);
9:    function hasMember(address user) external view returns (bool);
22:    constructor() {
36:    function messageForTransferRestriction(uint8 restrictionCode) public view returns (string memory) {
45:    function member(address user) public view {
49:    function hasMember(address user) public view returns (bool) {
57:    function updateMember(address user, uint256 validUntil) public auth {
62:    function updateMembers(address[] memory users, uint256 validUntil) public auth {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol

```solidity
8:interface IERC1271 {
9:    function isValidSignature(bytes32, bytes memory) external view returns (bytes4);
36:    event Rely(address indexed user);
37:    event Deny(address indexed user);
38:    event File(bytes32 indexed what, string data);
39:    event Approval(address indexed owner, address indexed spender, uint256 value);
40:    event Transfer(address indexed from, address indexed to, uint256 value);
42:    constructor(uint8 decimals_) {
51:    modifier auth() {
57:    function rely(address user) external auth {
62:    function deny(address user) external auth {
67:    function _calculateDomainSeparator(uint256 chainId) private view returns (bytes32) {
79:    function DOMAIN_SEPARATOR() external view returns (bytes32) {
83:    function file(bytes32 what, string memory data) external auth {
91:    function transfer(address to, uint256 value) public virtual returns (bool) {
106:    function transferFrom(address from, address to, uint256 value) public virtual returns (bool) {
131:    function approve(address spender, uint256 value) external returns (bool) {
139:    function increaseAllowance(address spender, uint256 addedValue) external returns (bool) {
148:    function decreaseAllowance(address spender, uint256 subtractedValue) external returns (bool) {
162:    function mint(address to, uint256 value) public virtual auth {
172:    function burn(address from, uint256 value) external auth {
196:    function _isValidSignature(address signer, bytes32 digest, bytes memory signature) internal view returns (bool) {
216:    function permit(address owner, address spender, uint256 value, uint256 deadline, bytes memory signature) public {
239:    function permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/Tranche.sol

```solidity
7:  interface TrancheTokenLike is IERC20 {
8:    function file(bytes32 what, string memory data) external;
9:    function restrictionManager() external view returns (address);
12:  interface ERC1404Like {
13:    function detectTransferRestriction(address from, address to, uint256 value) external view returns (uint8);
14:    function messageForTransferRestriction(uint8 restrictionCode) external view returns (string memory);
15:    function SUCCESS_CODE() external view returns (uint8);
29:    event File(bytes32 indexed what, address data);
30:    event AddLiquidityPool(address indexed liquidityPool);
31:    event RemoveLiquidityPool(address indexed liquidityPool);
33:    constructor(uint8 decimals_) ERC20(decimals_) {}
35:    modifier restricted(address from, address to, uint256 value) {
42:    function file(bytes32 what, address data) public auth {
48:    function addLiquidityPool(address liquidityPool) public auth {
53:    function removeLiquidityPool(address liquidityPool) public auth {
59:    function transfer(address to, uint256 value) public override restricted(_msgSender(), to, value) returns (bool) {
63:    function transferFrom(address from, address to, uint256 value)
72:    function mint(address to, uint256 value) public override restricted(_msgSender(), to, value) {
76:    function detectTransferRestriction(address from, address to, uint256 value) public view returns (uint8) {
80:    function checkTransferRestriction(address from, address to, uint256 value) public view returns (bool) {
84:    function messageForTransferRestriction(uint8 restrictionCode) public view returns (string memory) {
88:    function SUCCESS_CODE() public view returns (uint8) {
93:    function isTrustedForwarder(address forwarder) public view returns (bool) {
103:    function _msgSender() internal view virtual override returns (address sender) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/DelayedAdmin.sol

```solidity
16:    event File(bytes32 indexed what, address indexed data);
18:    constructor(address root_) {
26:    function pause() public auth {
30:    function unpause() public auth {
34:    function scheduleRely(address target) public auth {
38:    function cancelRely(address target) public auth {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/admins/PauseAdmin.sol

```solidity
15:    event AddPauser(address indexed user);
16:    event RemovePauser(address indexed user);
19:    event File(bytes32 indexed what, address indexed data);
21:    constructor(address root_) {
28:    modifier canPause() {
34:    function addPauser(address user) external auth {
39:    function removePauser(address user) external auth {
45:    function pause() public canPause {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol

```solidity
6:interface AuthLike {
7:    function rely(address) external;
8:    function deny(address) external;
26:    event File(bytes32 indexed what, uint256 data);
27:    event Pause();
28:    event Unpause();
29:    event RelyScheduled(address indexed target, uint256 indexed scheduledTime);
30:    event RelyCancelled(address indexed target);
31:    event RelyContract(address target, address indexed user);
32:    event DenyContract(address target, address indexed user);
34:    constructor(address _escrow, uint256 _delay) {
43:    function file(bytes32 what, uint256 data) external auth {
54:    function pause() external auth {
59:    function unpause() external auth {
65:    function scheduleRely(address target) external auth {
70:    function cancelRely(address target) external auth {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol

```solidity
7:  interface ERC20Like {
8:    function allowance(address owner, address spender) external view returns (uint256);
20:    event TransferIn(address indexed token, address indexed source, address indexed destination, uint256 amount);
21:    event TransferOut(address indexed token, address indexed destination, uint256 amount);
23:    constructor() {
29:    function transferIn(address token, address source, address destination, uint256 amount) external auth {
36:    function transferOut(address token, address destination, address receiver, uint256 amount) external auth {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Escrow.sol

```solidity
7:  interface ApproveLike {
8:    function approve(address, uint256) external returns (bool);
15:    event Approve(address indexed token, address indexed spender, uint256 value);
17:    constructor() {
23:    function approve(address token, address spender, uint256 value) external auth {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/PoolManager.sol

```solidity
11:  interface GatewayLike {
12:    function transferTrancheTokensToCentrifuge(
19:    function transferTrancheTokensToEVM(
27:    function transfer(uint128 currency, address sender, bytes32 recipient, uint128 amount) external;
30:interface LiquidityPoolLike {
31:    function hasMember(address) external returns (bool);
34:interface InvestmentManagerLike {
35:    function liquidityPools(uint64 poolId, bytes16 trancheId, address currency) external returns (address);
36:    function getTrancheToken(uint64 _poolId, bytes16 _trancheId) external view returns (address);
37:    function userEscrow() external view returns (address);
40:interface EscrowLike {
41:    function approve(address token, address spender, uint256 value) external;
44:interface ERC2771Like {
45:    function addLiquidityPool(address forwarder) external;
48:interface AuthLike {
49:    function rely(address usr) external;
53:struct Pool {
61:struct Tranche {

95:    event File(bytes32 indexed what, address data);
96:    event PoolAdded(uint64 indexed poolId);
97:    event PoolCurrencyAllowed(uint128 indexed currency, uint64 indexed poolId);
98:    event TrancheAdded(uint64 indexed poolId, bytes16 indexed trancheId);
99:    event TrancheDeployed(uint64 indexed poolId, bytes16 indexed trancheId, address indexed token);
100:    event CurrencyAdded(uint128 indexed currency, address indexed currencyAddress);
101:    event LiquidityPoolDeployed(uint64 indexed poolId, bytes16 indexed trancheId, address indexed liquidityPoool);
102:    event TrancheTokenDeployed(uint64 indexed poolId, bytes16 indexed trancheId);
104:    constructor(address escrow_, address liquidityPoolFactory_, address trancheTokenFactory_) {
120:    function file(bytes32 what, address data) external auth {
128:    function transfer(address currencyAddress, bytes32 recipient, uint128 amount) public {
136:    function transferTrancheTokensToCentrifuge(
149:    function transferTrancheTokensToEVM(
168:    function addPool(uint64 poolId) public onlyGateway {
179:    function allowPoolCurrency(uint64 poolId, uint128 currency) public onlyGateway {
192:    function addTranche(
214:    function updateTrancheTokenMetadata(
227:    function updateMember(uint64 poolId, bytes16 trancheId, address user, uint64 validUntil) public onlyGateway {
238:    function addCurrency(uint128 currency, address currencyAddress) public onlyGateway {
257:    function handleTransfer(uint128 currency, address recipient, uint128 amount) public onlyGateway {
265:    function handleTransferTrancheTokens(uint64 poolId, bytes16 trancheId, address destinationAddress, uint128 amount)
280:    function deployTranche(uint64 poolId, bytes16 trancheId) public returns (address) {
307:    function deployLiquidityPool(uint64 poolId, bytes16 trancheId, address currency) public returns (address) {
336:    function getTrancheToken(uint64 poolId, bytes16 trancheId) public view returns (address) {
341:    function getLiquidityPool(uint64 poolId, bytes16 trancheId, address currency) public view returns (address) {
345:    function isAllowedAsPoolCurrency(uint64 poolId, address currencyAddress) public view returns (bool) {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/InvestmentManager.sol

```solidity
8:  interface GatewayLike {
9:    function increaseInvestOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency, uint128 amount)
11:    function decreaseInvestOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency, uint128 amount)
13:    function increaseRedeemOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency, uint128 amount)
15:    function decreaseRedeemOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency, uint128 amount)
17:    function collectInvest(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
18:    function collectRedeem(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
19:    function cancelInvestOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
20:    function cancelRedeemOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency) external;
23:interface ERC20Like {
24:    function approve(address token, address spender, uint256 value) external;
25:    function transferFrom(address from, address to, uint256 amount) external returns (bool);
26:    function decimals() external view returns (uint8);
27:    function mint(address, uint256) external;
28:    function burn(address, uint256) external;
31:interface LiquidityPoolLike is ERC20Like {
32:    function poolId() external returns (uint64);
33:    function trancheId() external returns (bytes16);
34:    function asset() external view returns (address);
35:    function hasMember(address) external returns (bool);
36:    function updatePrice(uint128 price) external;
37:    function checkTransferRestriction(address from, address to, uint256 value) external view returns (bool);
38:    function latestPrice() external view returns (uint128);
41:interface PoolManagerLike {
42:    function currencyIdToAddress(uint128 currencyId) external view returns (address);
43:    function currencyAddressToId(address addr) external view returns (uint128);
44:    function getTrancheToken(uint64 poolId, bytes16 trancheId) external view returns (address);
45:    function getLiquidityPool(uint64 poolId, bytes16 trancheId, address currency) external view returns (address);
46:    function isAllowedAsPoolCurrency(uint64 poolId, address currencyAddress) external view returns (bool);
49:interface EscrowLike {
50:    function approve(address token, address spender, uint256 value) external;
53:interface UserEscrowLike {
54:    function transferIn(address token, address source, address destination, uint256 amount) external;
55:    function transferOut(address token, address owner, address destination, uint256 amount) external;
84:    event File(bytes32 indexed what, address data);
85:    event DepositProcessed(address indexed liquidityPool, address indexed user, uint128 indexed currencyAmount);
86:    event RedemptionProcessed(address indexed liquidityPool, address indexed user, uint128 indexed trancheTokenAmount);
88:    constructor(address escrow_, address userEscrow_) {
103:    function file(bytes32 what, address data) external auth {

// @param / @return missing
117:    function requestDeposit(uint256 currencyAmount, address user) public auth {
148:    function requestRedeem(uint256 trancheTokenAmount, address user) public auth {
174:    function decreaseDepositRequest(uint256 _currencyAmount, address user) public auth {
187:    function decreaseRedeemRequest(uint256 _trancheTokenAmount, address user) public auth {
200:    function collectDeposit(address user) public auth {
211:    function collectRedeem(address user) public auth {
223:    function updateTrancheTokenPrice(uint64 poolId, bytes16 trancheId, uint128 currencyId, uint128 price)
234:    function handleExecutedCollectInvest(
255:    function handleExecutedCollectRedeem(
277:    function handleExecutedDecreaseInvestOrder(
294:    function handleExecutedDecreaseRedeemOrder(
319:    function totalAssets(uint256 totalSupply, address liquidityPool) public view returns (uint256 _totalAssets) {
325:    function convertToShares(uint256 _assets, address liquidityPool) public view auth returns (uint256 shares) {
338:    function convertToAssets(uint256 _shares, address liquidityPool) public view auth returns (uint256 assets) {
350:    function maxDeposit(address user, address liquidityPool) public view returns (uint256 currencyAmount) {
355:    function maxMint(address user, address liquidityPool) public view returns (uint256 trancheTokenAmount) {
360:    function maxWithdraw(address user, address liquidityPool) public view returns (uint256 currencyAmount) {
365:    function maxRedeem(address user, address liquidityPool) public view returns (uint256 trancheTokenAmount) {
370:    function previewDeposit(address user, address liquidityPool, uint256 _currencyAmount)
383:    function previewMint(address user, address liquidityPool, uint256 _trancheTokenAmount)
396:    function previewWithdraw(address user, address liquidityPool, uint256 _currencyAmount)
409:    function previewRedeem(address user, address liquidityPool, uint256 _trancheTokenAmount)
427:    function processDeposit(address user, uint256 currencyAmount) public auth returns (uint256 trancheTokenAmount) {
451:    function processMint(address user, uint256 trancheTokenAmount) public auth returns (uint256 currencyAmount) {
467:    function _deposit(uint128 trancheTokenAmount, uint128 currencyAmount, address liquidityPool, address user)
489:    function processRedeem(uint256 trancheTokenAmount, address receiver, address user)
515:    function processWithdraw(uint256 currencyAmount, address receiver, address user)
535:    function _redeem(
551:    function calculateDepositPrice(address user, address liquidityPool) public view returns (uint256 depositPrice) {
560:    function calculateRedeemPrice(address user, address liquidityPool) public view returns (uint256 redeemPrice) {
569:    function _calculatePrice(uint128 currencyAmount, uint128 trancheTokenAmount, address liquidityPool)
584:    function _updateLiquidityPoolPrice(address liquidityPool, uint128 currencyPayout, uint128 trancheTokensPayout)
591:    function _calculateTrancheTokenAmount(uint128 currencyAmount, address liquidityPool, uint256 price)
605:    function _calculateCurrencyAmount(uint128 trancheTokenAmount, address liquidityPool, uint256 price)
619:    function _decreaseDepositLimits(address user, address liquidityPool, uint128 _currency, uint128 trancheTokens)
635:    function _decreaseRedemptionLimits(address user, address liquidityPool, uint128 _currency, uint128 trancheTokens)
651:    function _isAllowedToInvest(uint64 poolId, bytes16 trancheId, address currency, address user)
666:    function _toUint128(uint256 _value) internal pure returns (uint128 value) {
676:    function _toPriceDecimals(uint128 _value, uint8 decimals, address liquidityPool)
686:    function _fromPriceDecimals(uint256 _value, uint8 decimals, address liquidityPool)
696:    function _getPoolDecimals(address liquidityPool)
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/LiquidityPool.sol

```solidity
9:  interface ERC20PermitLike {
10:    function permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
12:    function PERMIT_TYPEHASH() external view returns (bytes32);
13:    function DOMAIN_SEPARATOR() external view returns (bytes32);
16:  interface TrancheTokenLike is IERC20, ERC20PermitLike {
17:    function checkTransferRestriction(address from, address to, uint256 value) external view returns (bool);
20:  interface InvestmentManagerLike {
21:    function processDeposit(address receiver, uint256 assets) external returns (uint256);
22:    function processMint(address receiver, uint256 shares) external returns (uint256);
23:    function processWithdraw(uint256 assets, address receiver, address owner) external returns (uint256);
24:    function processRedeem(uint256 shares, address receiver, address owner) external returns (uint256);
25:    function maxDeposit(address user, address _tranche) external view returns (uint256);
26:    function maxMint(address user, address _tranche) external view returns (uint256);
27:    function maxWithdraw(address user, address _tranche) external view returns (uint256);
28:    function maxRedeem(address user, address _tranche) external view returns (uint256);
29:    function totalAssets(uint256 totalSupply, address liquidityPool) external view returns (uint256);
30:    function convertToShares(uint256 assets, address liquidityPool) external view returns (uint256);
31:    function convertToAssets(uint256 shares, address liquidityPool) external view returns (uint256);
32:    function previewDeposit(address user, address liquidityPool, uint256 assets) external view returns (uint256);
33:    function previewMint(address user, address liquidityPool, uint256 shares) external view returns (uint256);
34:    function previewWithdraw(address user, address liquidityPool, uint256 assets) external view returns (uint256);
35:    function previewRedeem(address user, address liquidityPool, uint256 shares) external view returns (uint256);
36:    function requestRedeem(uint256 shares, address receiver) external;
37:    function requestDeposit(uint256 assets, address receiver) external;
38:    function collectDeposit(address receiver) external;
39:    function collectRedeem(address receiver) external;
40:    function decreaseDepositRequest(uint256 assets, address receiver) external;
41:    function decreaseRedeemRequest(uint256 shares, address receiver) external;
78:    event File(bytes32 indexed what, address data);
79:    event DepositRequested(address indexed owner, uint256 assets);
80:    event RedeemRequested(address indexed owner, uint256 shares);
81:    event DepositCollected(address indexed owner);
82:    event RedeemCollected(address indexed owner);
83:    event UpdatePrice(uint128 price);
85:    constructor(uint64 poolId_, bytes16 trancheId_, address asset_, address share_, address investmentManager_) {

// @param / @return missing
97:    modifier withApproval(address owner) {
103:    function file(bytes32 what, address data) public auth {
118:    function convertToShares(uint256 assets) public view returns (uint256 shares) {
125:    function convertToAssets(uint256 shares) public view returns (uint256 assets) {
130:    function maxDeposit(address receiver) public view returns (uint256) {
135:    function previewDeposit(uint256 assets) public view returns (uint256 shares) {
141:    function deposit(uint256 assets, address receiver) public returns (uint256 shares) {
148:    function mint(uint256 shares, address receiver) public returns (uint256 assets) {
155:    function maxMint(address receiver) external view returns (uint256 maxShares) {
160:    function previewMint(uint256 shares) external view returns (uint256 assets) {
165:    function maxWithdraw(address receiver) public view returns (uint256 maxAssets) {
170:    function previewWithdraw(uint256 assets) public view returns (uint256 shares) {
176:    function withdraw(uint256 assets, address receiver, address owner)
187:    function maxRedeem(address owner) public view returns (uint256 maxShares) {
192:    function previewRedeem(uint256 shares) public view returns (uint256 assets) {
200:    function redeem(uint256 shares, address receiver, address owner)
214:    function requestDeposit(uint256 assets, address owner) public withApproval(owner) {
220:    function requestDepositWithPermit(uint256 assets, address owner, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
231:    function requestRedeem(uint256 shares, address owner) public withApproval(owner) {
237:    function requestRedeemWithPermit(uint256 shares, address owner, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
247:    function decreaseDepositRequest(uint256 assets, address owner) public withApproval(owner) {
253:    function decreaseRedeemRequest(uint256 shares, address owner) public withApproval(owner) {
259:    function collectDeposit(address receiver) public {
265:    function collectRedeem(address receiver) public {
271:    function name() public view returns (string memory) {
275:    function symbol() public view returns (string memory) {
279:    function decimals() public view returns (uint8) {
283;    function totalSupply() public view returns (uint256) {
287:    function balanceOf(address owner) public view returns (uint256) {
291:    function allowance(address owner, address spender) public view returns (uint256) {
295:    function transferFrom(address, address, uint256) public returns (bool) {
301:    function transfer(address, uint256) public returns (bool) {
307:    function approve(address, uint256) public returns (bool) {
313:    function mint(address, uint256) public auth {
318:    function burn(address, uint256) public auth {
324:    function updatePrice(uint128 price) public auth {
332:    function checkTransferRestriction(address from, address to, uint256 value) public view returns (bool) {
338:    function _successCheck(bool success) internal pure {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/SafeTransferLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
15:    function safeTransferFrom(address token, address from, address to, uint256 value) internal {
26:    function safeTransfer(address token, address to, uint256 value) internal {
36:    function safeApprove(address token, address to, uint256 value) internal {
```

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/gateway/Messages.sol

```solidity
// private and internal `functions` should preppend with `underline`
68:    function messageType(bytes memory _msg) internal pure returns (Call _call) {
79:    function formatAddCurrency(uint128 currency, address currencyAddress) internal pure returns (bytes memory) {
68:    function messageType(bytes memory _msg) internal pure returns (Call _call) {
83:    function isAddCurrency(bytes memory _msg) internal pure returns (bool) {
87:    function parseAddCurrency(bytes memory _msg) internal pure returns (uint128 currency, address currencyAddress) {
102:    function isAddPool(bytes memory _msg) internal pure returns (bool) {
106:    function parseAddPool(bytes memory _msg) internal pure returns (uint64 poolId) {
121:    function isAllowPoolCurrency(bytes memory _msg) internal pure returns (bool) {
125:    function parseAllowPoolCurrency(bytes memory _msg) internal pure returns (uint64 poolId, uint128 currency) {
163:    function isAddTranche(bytes memory _msg) internal pure returns (bool) {
167:    function parseAddTranche(bytes memory _msg)
205:    function formatUpdateMember(uint64 poolId, bytes16 trancheId, bytes32 member, uint64 validUntil)
213:    function isUpdateMember(bytes memory _msg) internal pure returns (bool) {
217:    function parseUpdateMember(bytes memory _msg)
245:    function isUpdateTrancheTokenPrice(bytes memory _msg) internal pure returns (bool) {
249:    function parseUpdateTrancheTokenPrice(bytes memory _msg)
277:    function isTransfer(bytes memory _msg) internal pure returns (bool) {
281:    function parseTransfer(bytes memory _msg)
294:    function parseIncomingTransfer(bytes memory _msg)
345:    function isTransferTrancheTokens(bytes memory _msg) internal pure returns (bool) {
351:    function parseTransferTrancheTokens20(bytes memory _msg)
384:    function isIncreaseInvestOrder(bytes memory _msg) internal pure returns (bool) {
388:    function parseIncreaseInvestOrder(bytes memory _msg)
420:    function isDecreaseInvestOrder(bytes memory _msg) internal pure returns (bool) {
424:    function parseDecreaseInvestOrder(bytes memory _msg)
452:    function isIncreaseRedeemOrder(bytes memory _msg) internal pure returns (bool) {
456:    function parseIncreaseRedeemOrder(bytes memory _msg)
484:    function isDecreaseRedeemOrder(bytes memory _msg) internal pure returns (bool) {
488:    function parseDecreaseRedeemOrder(bytes memory _msg)
512:    function isCollectInvest(bytes memory _msg) internal pure returns (bool) {
516:    function parseCollectInvest(bytes memory _msg)
543:    function isCollectRedeem(bytes memory _msg) internal pure returns (bool) {
547:    function parseCollectRedeem(bytes memory _msg)
558:    function formatExecutedDecreaseInvestOrder(
570:    function isExecutedDecreaseInvestOrder(bytes memory _msg) internal pure returns (bool) {
574:    function parseExecutedDecreaseInvestOrder(bytes memory _msg)
586:    function formatExecutedDecreaseRedeemOrder(
598:    function isExecutedDecreaseRedeemOrder(bytes memory _msg) internal pure returns (bool) {
602:    function parseExecutedDecreaseRedeemOrder(bytes memory _msg)
614:    function formatExecutedCollectInvest(
633:    function isExecutedCollectInvest(bytes memory _msg) internal pure returns (bool) {
637:    function parseExecutedCollectInvest(bytes memory _msg)
657:    function formatExecutedCollectRedeem(
676:    function isExecutedCollectRedeem(bytes memory _msg) internal pure returns (bool) {
680:    function parseExecutedCollectRedeem(bytes memory _msg)
700:    function parseExecutedCollectRedeem(bytes memory _msg)
704:    function isScheduleUpgrade(bytes memory _msg) internal pure returns (bool) {
708:    function parseScheduleUpgrade(bytes memory _msg) internal pure returns (address _contract) {
712:    function formatCancelUpgrade(address _contract) internal pure returns (bytes memory) {
716:    function isCancelUpgrade(bytes memory _msg) internal pure returns (bool) {
720:    function parseCancelUpgrade(bytes memory _msg) internal pure returns (address _contract) {
751:    function isUpdateTrancheTokenMetadata(bytes memory _msg) internal pure returns (bool) {
755:    function parseUpdateTrancheTokenMetadata(bytes memory _msg)
766:    function formatCancelInvestOrder(uint64 poolId, bytes16 trancheId, bytes32 investor, uint128 currency)
774:    function parseCancelInvestOrder(bytes memory _msg)
785:    function formatCancelRedeemOrder(uint64 poolId, bytes16 trancheId, bytes32 investor, uint128 currency)
793:    function parseCancelRedeemOrder(bytes memory _msg)
820:    function isUpdateTrancheInvestmentLimit(bytes memory _msg) internal pure returns (bool) {
824:    function parseUpdateTrancheInvestmentLimit(bytes memory _msg)
836:    function formatDomain(Domain domain) public pure returns (bytes9) {
840:    function formatDomain(Domain domain, uint64 chainId) public pure returns (bytes9) {
```

---


### Initialized variables [5]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol

```solidity
23:    bool public paused = false;
```

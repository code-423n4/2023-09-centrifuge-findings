# Approach taken in evaluating the codebase

`Steps:`

- `Use a static code analysis tool:` Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

  - Insecure coding practices
  - Common vulnerabilities
  - Code that is not compliant with security standards

- `Read the documentation:` The documentation for Centrifuge pools should provide a detailed overview of the Asset Originators and its codebase. This documentation can be used to understand the purpose of the code and to identify potential areas of concern.

- `Scope the analysis:` Once you have a basic understanding of the protocol and its codebase, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external APIs, or the code that stores sensitive data.

- `Manually review the code:` Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

  - Unvalidated user input
  - Hardcoded credentials
  - Insecure cryptographic functions
  - Unsafe deserialization

- `Mark vulnerable code parts with @audit tags:` Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on. 

- `Dig deep into vulnerable code parts and compare with documentations:` For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- `Perform a series of tests:` Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:
  - Valid and invalid user input
  - Different types of attacks
  - Different operating systems and hardware platforms

- `Report any problems:` If you find any problems with the code, you should report them to the developers of Centrifuge pools. The developers will then be able to fix the problems and release a new version of the protocol.

# Architecture Recommendations:

- `Security Audits:` Conduct thorough security audits by experts to identify and mitigate vulnerabilities.

- `Robust Authorization:` Implement strong authorization mechanisms for functions to ensure that only authorized entities can access critical functions.

- `Input Validation:` Implement rigorous input validation to prevent malicious inputs.

- `Gas Optimization:` Ensure functions are gas-efficient to avoid transaction failures.

- `Documentation:` Provide detailed documentation to aid developers and auditors.

- `Reentrancy Protection:` Be cautious with external contract calls and ensure reentrancy vulnerabilities are addressed.

- `Liquidity Management:` Effectively manage pool liquidity to optimize yields and minimize cash drag.

- `Reserve Management:` Carefully define the max reserve amount to control liquidity.

- `Subordination Ratios:` Monitor subordination ratios to maintain risk protection.

- `Regular Updates:` Stay informed about changes in financing fees, CFG rewards, and risk metrics.

# Codebase quality analysis

`src/admins/DelayedAdmin.sol`
  - The contract uses the OpenZeppelin library for safe token transfers: This is not directly visible in the provided code, but it's mentioned in the import statement on line 5. The SafeTransferLib is assumed to be a part of the OpenZeppelin library, which is known for its secure and community-vetted contracts.

  - The approve function is protected by the auth modifier: This is visible on line 21. The auth modifier is likely defined in the Auth contract that is inherited by the Escrow contract. This modifier restricts the approve function to be called only by addresses that have been authorized (wards).

  - The contract emits events for important actions: This is visible on line 13 and line 23. The Approve event is emitted whenever the approve function is called, providing transparency and making it easier to track the contract's activities off-chain.

`src/admins/PauseAdmin.sol`
  - The contract assumes that the Root contract's pause() function is secure and does not validate any conditions before calling it. If the Root contract's pause() function has any vulnerabilities, they would be propagated here.

  - The contract does not prevent zero addresses (0x0) from being added as pausers. This could lead to potential misuse or errors.

  - The contract does not check if a pauser already exists before adding or if a pauser does not exist before removing. This could lead to unexpected behavior.

  - The contract does not have a function to renounce admin rights. If the admin's private key is compromised, there's no way to renounce the admin rights to prevent misuse.

  - The contract does not emit an event when the pause() function is called. This could make tracking of state changes harder.  

`src/gateway/Gateway.sol`

  - Line 47: The constructor does not check if the addresses of root_, investmentManager_, poolManager_, and router_ are non-zero addresses. This could lead to a potential risk if the contract is initialized with zero addresses.

  - Line 69-78: The file function allows the owner to change the poolManager and investmentManager addresses. If the owner's private key is compromised, an attacker could change these addresses to malicious contracts.

  - Line 80-88: The addIncomingRouter, removeIncomingRouter, and updateOutgoingRouter functions allow the owner to change the router addresses. If the owner's private key is compromised, an attacker could change these addresses to malicious contracts.

  - Line 90-246: The contract does not check if the msg.sender is a non-zero address in the onlyInvestmentManager, onlyPoolManager, and onlyIncomingRouter modifiers. This could lead to potential risks if these functions are called by a zero address.

  - Line 248-370: The handle function does not validate the incoming message. This could lead to potential risks if the function is called with an invalid message.

  - Line 372: The _addressToBytes32 function does not check if the input address is a non-zero address. This could lead to potential risks if the function is called with a zero address.


`src/gateway/routers/axelar/Router.sol`

  - Line 24: The AxelarGatewayLike and GatewayLike interfaces are imported without any checks on the functions they provide. If these interfaces are not implemented correctly, it could lead to unexpected behavior.

  - Line 38: The onlyCentrifugeChainOrigin modifier checks if the msg.sender is the axelarGateway and if the sourceChain and sourceAddress match the expected values. However, it does not check if the axelarGateway itself is a trusted contract. If the axelarGateway is compromised, it could lead to unauthorized function calls.

  - Line 49: The onlyGateway modifier checks if the msg.sender is the gateway. Again, it does not check if the gateway is a trusted contract. If the gateway is compromised, it could lead to unauthorized function calls.

  - Line 59: The file function allows to change the gateway address. If an unauthorized user gains access to this function, they could change the gateway to a malicious contract.

  - Line 73: The execute function calls the handle function of the gateway contract with the provided payload. If the handle function of the gateway contract is not implemented correctly, it could lead to reentrancy attacks or other vulnerabilities.

  - Line 82: The send function calls the callContract function of the axelarGateway contract. If the callContract function of the axelarGateway contract is not implemented correctly, it could lead to reentrancy attacks or other vulnerabilities.

`src/token/ERC20.sol`

 - Potential Integer Overflow/Underflow:
    The unchecked keyword is used in several places throughout the code, such as in the transfer and transferFrom functions.

 - Lack of Function Input Validation:
    The transfer function: function transfer(address to, uint256 value) public virtual returns (bool)

 - The transferFrom function: function transferFrom(address from, address to, uint256 value) public virtual returns (bool)

 - Potential Reentrancy Vulnerability:
    Any external or fallback function calls made within the contract could potentially introduce reentrancy vulnerabilities. The presence of such calls is not evident from the provided code snippet alone.

`src/token/RestrictionManager.sol`

   - Line 23: The constructor sets the contract deployer as an authorized user. If the deployer's private key is compromised, the attacker can update the member list.

   - Line 38: The detectTransferRestriction function does not check if the from address is a member. This could potentially allow non-members to initiate transfers.

   - Line 52: The member function does not return any value. It only throws an error if the user is not a member. This could be confusing for developers who expect it to return a boolean.

   - Line 59: The hasMember function checks if the membership of a user is not expired. However, it does not check if the user is a member in the first place.

   - Line 66: The updateMember function allows an authorized user to extend the membership of a user indefinitely. This could potentially be abused.

   - Line 73: The updateMembers function allows an authorized user to update multiple members at once. If the array of users is too large, this could potentially cause the transaction to run out of gas.

`src/token/Tranche.sol`

  - Line 33: The restricted modifier uses the detectTransferRestriction function to check if a transfer is allowed. However, the detectTransferRestriction function is not defined in this contract and is instead called from the restrictionManager contract. If the restrictionManager contract is compromised, it could allow unauthorized transfers.

  - Line 43: The file function allows the restrictionManager contract to be changed. If an attacker gains control of an account with the auth modifier, they could change the restrictionManager to a malicious contract.

  - Line 49 and 54: The addLiquidityPool and removeLiquidityPool functions can be called by any account with the auth modifier. If an attacker gains control of such an account, they could add or remove liquidity pools at will.

  - Line 70, 76, and 82: The transfer, transferFrom, and mint functions use the restricted modifier, which relies on the restrictionManager contract. If the restrictionManager contract is compromised, it could allow unauthorized transfers or minting of tokens.

  - Line 97: The isTrustedForwarder function trusts any address that is a liquidity pool. If an attacker can add a malicious contract as a liquidity pool, they could exploit this trust.

  - Line 104: The _msgSender function trusts the msg.sender if it is a trusted forwarder. If an attacker can add a malicious contract as a trusted forwarder, they could exploit this trust.

`src/util/Auth.sol`

  - Authorization control: The rely and deny functions can be called by anyone who is currently a ward. This could potentially lead to misuse if a ward is compromised. A potential mitigation could be to have a separate role for administrative control.

  - No input validation: The rely and deny functions do not check if the input address is a zero address. This could lead to potential misuse or bugs.

  - No event emitted on failure: If the auth modifier fails, it only reverts the transaction without emitting any event. This could make it harder to debug issues.

`src/util/Factory.sol`

  - Lack of input validation: This issue is present in the newLiquidityPool function (lines 30-41) of the LiquidityPoolFactory contract and the newTrancheToken function (lines 30-50) of the TrancheTokenFactory contract. There are no checks to validate the input parameters in these functions.

  - Duplicate addresses in wards array: This issue is present in the newLiquidityPool function (lines 37-38) of the LiquidityPoolFactory contract and the newTrancheToken function (lines 45-46) of the TrancheTokenFactory contract. There are no checks to validate if the wards array contains duplicate addresses.

  - No events emitted when new contracts are created: This issue is present in the newLiquidityPool function (line 41) of the LiquidityPoolFactory contract and the newTrancheToken function (line 50) of the TrancheTokenFactory contract. There are no events emitted when new contracts are created.

  - No check for zero root address: This issue is present in the constructor of both LiquidityPoolFactory (lines 20-24) and TrancheTokenFactory (lines 30-34) contracts. There are no checks to validate if the root address is a zero address.  

`src/Escrow.sol`

  - The approve function is protected by the auth modifier (line 21). If the Auth contract is not implemented correctly, unauthorized addresses could potentially call this function.


  - The contract doesn't check if the approve function of the ApproveLike interface (line 7) returns true. If the function returns false, the approve function of the Escrow contract will still emit the Approve event (line 23), which could be misleading.

`src/LiquidityPool.sol`

  - Line 68: The investmentManager variable is public and can be changed by anyone with auth permissions. If the contract that has auth permissions gets compromised, the attacker can change the investmentManager to a malicious contract.

  - Line 68: The investmentManager variable is public and can be changed by anyone with auth permissions. If the contract that has auth permissions gets compromised, the attacker can change the investmentManager to a malicious contract. 

`src/PoolManager.sol`

  - The transfer function in the PoolManager contract doesn't have any access control. This means that anyone can call this function, which could lead to unauthorized transfers.

  - The file function in the PoolManager contract allows the gateway and investmentManager addresses to be changed by anyone with the auth role. If the auth role is compromised, this could lead to a serious security issue.

  - The addPool, allowPoolCurrency, addTranche, addCurrency, handleTransfer, and handleTransferTrancheTokens functions in the PoolManager contract are only protected by the onlyGateway modifier. If the gateway address is compromised, this could lead to serious security issues.

  - The deployTranche and deployLiquidityPool functions in the PoolManager contract are public and don't have any access control. This could potentially lead to unauthorized deployments.

  - The PoolManager contract relies heavily on external contracts such as gateway, investmentManager, escrow, liquidityPoolFactory, and trancheTokenFactory. If any of these contracts are compromised, it could lead to serious security issues in the PoolManager contract.

  - The PoolManager contract does not check if the escrow_, liquidityPoolFactory_, and trancheTokenFactory_ addresses provided in the constructor are valid (non-zero) addresses.

`src/Root.sol`

  - Line 24: The delay variable is public and can be set by anyone with auth permissions. If set to a very high value, it could potentially lock the contract's functionality for a long time. However, the code does have a check in place to ensure the delay cannot be more than 4 weeks.

  - Line 35: The wards mapping is used to manage permissions but it's not clear where it's defined or how it's managed. If not handled properly, this could lead to unauthorized access.

  - Line 57, 64, 71, 80, 87: All these functions are protected by the auth modifier. If the mechanism to assign and revoke auth permissions is not secure, it could lead to unauthorized access and manipulation of the contract.

  - Line 80 and 87: The relyContract and denyContract functions directly call external contracts. This could potentially be a security risk if the external contracts are not trusted or have bugs.

`src/UserEscrow.sol`
 - Line 28: The constructor sets msg.sender as a ward. If the contract is deployed by a factory contract, msg.sender will be the factory contract, not the EOA (Externally Owned Account) that initiated the transaction. This could lead to unexpected permission settings.

 - Line 34: The transferIn function does not check if the safeTransferFrom was successful. Although the SafeTransferLib should revert in case of a failed transfer, it would be safer to explicitly check the return value.

# Gas Optimization

`src/admins/DelayedAdmin.sol`
 the approve function (line 21) calls the SafeTransferLib.safeApprove function. This is an external call and consumes more gas than an internal call. If the safeApprove function is not complex and doesn't rely on the state of the SafeTransferLib contract, it might be more gas-efficient to implement this function directly in the Escrow contract.

 This change should not impact the rest of the code, as it only changes where the approve functionality is implemented. However, it's important to thoroughly test this change to ensure it doesn't introduce any new issues.

`src/gateway/Gateway.sol`

  - Use external instead of public for functions that are not called internally: Functions like addIncomingRouter, removeIncomingRouter, updateOutgoingRouter, transferTrancheTokensToCentrifuge, transferTrancheTokensToEVM, transfer, increaseInvestOrder, decreaseInvestOrder, increaseRedeemOrder, decreaseRedeemOrder, collectInvest, collectRedeem, cancelInvestOrder, cancelRedeemOrder, and handle are marked as public but they seem to be designed to be called from outside the contract. Marking these functions as external can save a little bit of gas because external functions have direct access to calldata.

  - Remove redundant code: The pauseable modifier is used in many functions to check if the contract is paused. However, this check is redundant in some functions because these functions are only called by the contract owner or specific addresses. You can remove the pauseable modifier from these functions to save gas.

  - Use uint256 instead of smaller integer types: In Solidity, smaller integer types like uint64 and uint128 do not save gas. In fact, they can sometimes be more expensive because the EVM has to use additional operations to manipulate the smaller types. Consider using uint256 for all integer variables and function parameters.

  - Optimize storage layout: Solidity stores contract state in 32-byte slots. You can pack smaller state variables into a single slot to save gas. For example, you can pack the incomingRouters mapping and the outgoingRouter address into a single slot because an address only takes up 20 bytes and a boolean only takes up 1 byte.

  - Use libraries for complex operations: The handle function contains a lot of complex logic. Consider moving this logic into a library to make the code more modular and easier to optimize.

  - Avoid unnecessary writes to storage: Writing to storage is one of the most expensive operations in Solidity. Avoid unnecessary writes by using local variables instead of state variables whenever possible. For example, in the file function, you can use a local variable to store the new address before checking if it's valid and writing it to storage.


`src/gateway/routers/axelar/Router.sol`

  - Use external instead of public for functions that are not called within the contract. In this contract, the execute and send functions could be declared as external.

       - Line 71: function execute(...) public onlyCentrifugeChainOrigin(sourceChain, sourceAddress) {...}
       - Line 80: function send(bytes calldata message) public onlyGateway {...}

  - Use events to store data instead of contract storage if the data is not needed for contract logic. In this contract, the File event is used to log changes to the gateway address, which is a good practice.
     - Line 66: emit File(what, data);

  - Use short-circuiting to your advantage. In Solidity, conditions in if statements and require functions are evaluated from left to right, and evaluation stops as soon as the result is determined. Therefore, put the most likely to fail or the cheapest conditions first. In this contract, the conditions in the onlyCentrifugeChainOrigin and onlyGateway modifiers are already ordered in a gas-efficient way.

    - Line 38-45: modifier onlyCentrifugeChainOrigin(string calldata sourceChain, string calldata sourceAddress) {...}

    - Line 47-50: modifier onlyGateway() {...}

`src/token/ERC20.sol`

  - Replace balanceOf[_msgSender()] with a local variable: In the transfer and transferFrom functions, the expression balanceOf[_msgSender()] is used multiple times. It's more gas-efficient to store this value in a local variable and use it throughout the function instead.

  - Use memory instead of storage for function parameters: In the isValidSignature function, the signature parameter can be declared as bytes memory instead of bytes memory. Using memory instead of storage can save gas costs.

  - Avoid unnecessary storage writes: In the burn function, the balanceOf[from] value can be stored in a local variable, and then it can be used to update balanceOf[from] and totalSupply. This avoids unnecessary storage writes and reduces gas consumption.

`src/token/RestrictionManager.sol`

  - Line 73-79: The updateMembers function iterates over an array of users to update their membership. This could be gas inefficient if the array is large. A possible optimization could be to limit the number of users that can be updated in a single transaction.

  - Line 66: The updateMember function updates the membership of a user every time it is called, even if the validUntil value is the same. You could add a condition to check if the new validUntil value is different from the current one before updating it to save gas.

  - Line 38: The detectTransferRestriction function checks if the to address is a member. This could be optimized by storing the result of hasMember(to) in a variable and reusing it, instead of calling the function twice.

  - Line 52: The member function uses a require statement to check if a user is a member. This could be optimized by replacing the require statement with a return statement, which would consume less gas in case of failure.

  - Line 59: The hasMember function checks if a user's membership is not expired. This could be optimized by storing the result of members[user] >= block.timestamp in a variable and reusing it, instead of performing the comparison twice.

  - Line 33: The restricted modifier calls the detectTransferRestriction function twice, once to check the restriction and once to get the message for the restriction. This could be optimized by storing the result in a variable and reusing it.

`src/token/Tranche.sol`

  - Line 43: The file function checks if the what parameter is equal to "restrictionManager". This could be optimized by using a bytes32 hash of the string instead of the string itself, as comparing hashes is cheaper than comparing strings.

  - Line 49 and 54: The addLiquidityPool and removeLiquidityPool functions emit events. If these functions are called frequently, it could be more gas efficient to batch these operations and emit a single event.

  - Line 70, 76, and 82: The transfer, transferFrom, and mint functions call the restricted modifier, which in turn calls the detectTransferRestriction function. If these functions are called frequently, it could be more gas efficient to batch these operations.

  - Line 97: The isTrustedForwarder function checks if an address is a liquidity pool. If this function is called frequently, it could be more gas efficient to use a more efficient data structure for storing and checking liquidity pools.

  - Line 104: The _msgSender function checks if the msg.sender is a trusted forwarder. If this function is called frequently, it could be more gas efficient to use a more efficient data structure for storing and checking trusted forwarders.


`src/Escrow.sol`

  - The approve function calls an external contract's function (SafeTransferLib.safeApprove). External calls are more gas-intensive than internal calls. If the safeApprove function could be implemented in this contract or if an equivalent function exists in this contract, it could be more gas-efficient to use that.

  - The approve function is only accessible to wards (authorized addresses). The auth modifier likely checks this condition, which consumes some gas. If the number of wards is small and doesn't change often, it might be more gas-efficient to hardcode these addresses.

`src/PoolManager.sol`

  - Use external instead of public for functions that are not called internally. For example, the deployTranche and deployLiquidityPool functions could potentially be marked as external.

  - Avoid unnecessary writes to storage. Writing to storage is one of the most expensive operations in terms of gas. For example, in the addPool function, you're writing to pool.poolId even though it's the same as the mapping key. This is unnecessary and wastes gas.


  - Avoid unnecessary contract interactions. Each external contract call consumes a significant amount of gas. For example, in the addCurrency function, you're calling approve on the escrow contract twice with the same parameters except for the spender. You could potentially combine these into a single external call to save gas.

  - Use libraries for common operations. For example, you're using SafeTransferLib for safe transfers, but you could also use libraries for other common operations like safe math.

  - Use events to reduce storage costs. If you're storing data that's not needed for contract logic but is only used for external tracking or information, consider using events instead.

  - Remove redundant code. For example, in the isAllowedAsPoolCurrency function, you're returning true at the end, but you've already required that the condition is true, so this is unnecessary.

`src/Root.sol`
  - Line 24: The delay variable is public, which means a getter function is automatically created. If this variable is not accessed externally, you could save gas by making it private.

  - Line 71: The executeScheduledRely function does not have an auth modifier. This means that anyone can call this function, potentially leading to unnecessary gas costs. Adding an auth modifier could prevent this.

  - Line 80 and 87: The relyContract and denyContract functions directly call external contracts. This could potentially be a gas cost if the external contracts are not optimized. You could save gas by optimizing these external calls.

  - General: Use events sparingly. Emitting events costs gas. If you have events that are not necessary for the functioning of your contract, consider removing them.  

`src/UserEscrow.sol`

  - Line 34 and 43: The transferIn and transferOut functions use the SafeTransferLib for transferring tokens. Depending on the implementation of this library, it might be more gas efficient to use the standard ERC20 transfer and transferFrom functions directly.

# Centralization risks
Centralization risks within this project are a matter of concern, as they pose a potential single point of failure. The centrality risk is elevated due to the critical powers vested in the onlyGateway and onlyInvestmentManager roles. The implications of these roles are far-reaching, and they must be handled with extreme caution. These roles, and their associated risks, can be summarized as follows:

onlyGateway: This role wields significant authority within the project. It holds the power to execute key functions that can directly impact the project and its financial resources. The integrity of this role is paramount because any compromise, whether through malicious intent or stolen private keys, could lead to the compromise of the entire project and its funds. The potential consequences include unauthorized fund transfers, manipulation of token prices, and disruption of essential operations.

onlyInvestmentManager: Similar to the onlyGateway role, the onlyInvestmentManager role also holds crucial powers. It is responsible for managing various aspects of investments and redemptions within the project. Compromised access to this role, whether by malicious actors or through private key theft, can result in unauthorized alterations to investment orders, withdrawals, or fund allocations. Such unauthorized actions could result in financial losses or the disruption of normal project operations.


`onlyGateway`
```solidity
File: src/InvestmentManager.sol
 
223    function updateTrancheTokenPrice(uint64 poolId, bytes16 trancheId, uint128 currencyId, uint128 price)
        public
        onlyGateway
    {

234     function handleExecutedCollectInvest(
        uint64 poolId,
        bytes16 trancheId,
        address recipient,
        uint128 currency,
        uint128 currencyPayout,
        uint128 trancheTokensPayout
    ) public onlyGateway {

255      function handleExecutedCollectRedeem(
        uint64 poolId,
        bytes16 trancheId,
        address recipient,
        uint128 currency,
        uint128 currencyPayout,
        uint128 trancheTokensPayout
    ) public onlyGateway {

277      function handleExecutedDecreaseInvestOrder(
        uint64 poolId,
        bytes16 trancheId,
        address user,
        uint128 currency,
        uint128 currencyPayout
    ) public onlyGateway {

294     function handleExecutedDecreaseRedeemOrder(
        uint64 poolId,
        bytes16 trancheId,
        address user,
        uint128 currency,
        uint128 trancheTokenPayout
    ) public onlyGateway {

File:  src/PoolManager.sol

168  function addPool(uint64 poolId) public onlyGateway {

179  function allowPoolCurrency(uint64 poolId, uint128 currency) public onlyGateway {

192      function addTranche(
        uint64 poolId,
        bytes16 trancheId,
        string memory tokenName,
        string memory tokenSymbol,
        uint8 decimals
    ) public onlyGateway {

214      function updateTrancheTokenMetadata(
        uint64 poolId,
        bytes16 trancheId,
        string memory tokenName,
        string memory tokenSymbol
    ) public onlyGateway {

227 function updateMember(uint64 poolId, bytes16 trancheId, address user, uint64 validUntil) public onlyGateway {

238  function addCurrency(uint128 currency, address currencyAddress) public onlyGateway {

257  function handleTransfer(uint128 currency, address recipient, uint128 amount) public onlyGateway {

265     function handleTransferTrancheTokens(uint64 poolId, bytes16 trancheId, address destinationAddress, uint128 amount)
        public
        onlyGateway
    {   


File: src/gateway/Gateway.sol
  onlyInvestmentManager
200      function increaseInvestOrder(
        uint64 poolId,
        bytes16 trancheId,
        address investor,
        uint128 currency,
        uint128 currencyAmount
    ) public onlyInvestmentManager pauseable {

212      function decreaseInvestOrder(
        uint64 poolId,
        bytes16 trancheId,
        address investor,
        uint128 currency,
        uint128 currencyAmount
    ) public onlyInvestmentManager pauseable {

224      function increaseRedeemOrder(
        uint64 poolId,
        bytes16 trancheId,
        address investor,
        uint128 currency,
        uint128 trancheTokenAmount
    ) public onlyInvestmentManager pauseable {

238      function decreaseRedeemOrder(
        uint64 poolId,
        bytes16 trancheId,
        address investor,
        uint128 currency,
        uint128 trancheTokenAmount
    ) public onlyInvestmentManager pauseable {

252      function collectInvest(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
        public
        onlyInvestmentManager
        pauseable
    {

260      function collectRedeem(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
        public
        onlyInvestmentManager
        pauseable
    {

268      function cancelInvestOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
        public
        onlyInvestmentManager
        pauseable
    {

276      function cancelRedeemOrder(uint64 poolId, bytes16 trancheId, address investor, uint128 currency)
        public
        onlyInvestmentManager
        pauseable
    {
```

# Other recommendations

`Enhance User Education:` Provide comprehensive educational resources for users to understand the platform's complexity, risks, and effective usage.

`Security Audits:` Regularly conduct comprehensive security audits to identify and address vulnerabilities in the smart contracts and platform.

`Transparency and Reporting:` Implement real-time reporting and auditing features for stakeholders to track pool performance and investments.

Community Governance: Consider decentralized governance mechanisms where token holders have a say in protocol decisions.

`Regulatory Compliance:` Stay informed about evolving regulatory requirements and seek legal counsel to ensure compliance.

`Diverse Asset Types:` Explore opportunities to expand the types of real-world assets that can be tokenized and financed through Centrifuge pools.


# Time spent on analysis
26 Hours

### Time spent:
26 hours
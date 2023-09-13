Divide/zero

division before multiplication

unsafe down casting 

Approving zero after normal amount

approving uint256.max reverts 

some tokens revert on zero amount and zero address

Contracts are vulnerable to fee-on-transfer accounting-related issues
The functions below transfer funds from the caller to the receiver via transferFrom(), but do not ensure that the actual number of tokens received is the same as the input amount to the transfer. If the token is a fee-on-transfer token, the balance after the transfer will be smaller than expected, leading to accounting issues. Even if there are checks later, related to a secondary transfer, an attacker may be able to use latent funds (e.g. mistakenly sent by another user) in order to get a free credit. One way to solve this problem is to measure the balance before and after the transfer, and use the difference as the amount, rather than the stated amount.

the decimal problems in tokens 

hash collision abi.encodepacked

push 0

solamate tokens transferlib need to check contract existence 

External calls in unbounded loops dos

zero address minting and transfers should be avoided 

Unsafe use of transfer()/transferFrom() with IERC20
Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions on L1 do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert (see this link for a test case). Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

return value of transfer and transferFrom functions not checked 

ecrecover address(0) checked 

need to use salt to avoid replying attacks 

is initialize functions front run ?

All upgradable contracts initialized

All hardcoded values correct also compatible for all chains ?

Is there any swap involved, hardcoded slippage, no deadline, or very less deadline 

Is this allowing (sender == Receiver )

The owner is a single point of failure and a centralization risk
Having a single EOA as the only owner of contracts is a large centralization risk and a single point of failure. A single private key may be taken in a hack, or the sole holder of the key may become unable to retrieve the key when necessary. Consider changing to a multi-signature setup, or having a role-based authorization model.

Excess funds sent via msg.value not refunded
The code below allows the caller to provide Ether, but does not refund the amount in excess of what's required, leaving funds stranded in the contract. The condition should be changed to check for equality, or the code should refund the excess.

Use of transferFrom() rather than safeTransferFrom() for NFTs in will lead to the loss of NFTs
The EIP-721 standard says the following about transferFrom():

 Return values of transfer()/transferFrom() not checked
Not all IERC20 implementations revert() when there's a failure in transfer()/transferFrom(). The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually making a payment

_safeMint() should be used rather than _mint() wherever possible
_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function. In the cases below, _mint() does not call ERC721TokenReceiver.onERC721Received() on the recipient.

 Some tokens may revert when zero value transfers are made
In spite of the fact that EIP-20 states that zero-valued transfers must be accepted, some tokens, such as LEND will revert if this is attempted, which may cause transactions that involve other tokens (such as batch operations) to fully revert. Consider skipping the transfer if the amount is zero, which will also save gas.

approve()/safeApprove() may revert if the current approval is not zero
Calling approve() without first calling approve(0) if the current approval is non-zero will revert with some tokens, such as Tether (USDT). While Tether is known to do this, it applies to other tokens as well, which are trying to protect against this attack vector. safeApprove() itself also implements this protection. Always reset the approval to zero before changing it to a new value, or use safeIncreaseAllowance()/safeDecreaseAllowance()

Multiplication on the result of a division
Dividing an integer by another integer will often result in loss of precision. When the result is multiplied by another number, the loss of precision is magnified, often to material magnitudes. (X / Z) * Y should be re-written as (X * Y) / Z

Division by zero not prevented
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

External calls in an un-bounded for-loop may result in a DOS
Consider limiting the number of iterations in for-loops that make external calls

Allowed fees/rates should be capped by smart contracts
Fees/rates should be required to be below 100%, preferably at a much lower limit, to ensure users don't have to monitor the blockchain for changes prior to using the protocol

Solidity version 0.8.20 may not work on other chains due to PUSH0
The compiler for Solidity 0.8.20 switches the default target EVM version to Shanghai, which includes the new PUSH0 op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier EVM version. While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

 Unsafe downcast
When a type is downcast to a smaller type, the higher order bits are truncated, effectively applying a modulo to the original value. Without any other checks, this wrapping will lead to unexpected behavior and bugs

Array lengths not checked
If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array

Empty receive()/payable fallback() function does not authorize requests
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

Draft imports may break in new minor versions
While OpenZeppelin draft contracts are safe to use and have been audited, their 'draft' status means that the EIPs they're based on are not finalized, and thus there may be breaking changes in even minor releases. If a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to has breaking changes in the new version, you'll encounter unnecessary delays in porting and testing replacement contracts. Ensure that you have extensive test coverage of this area so that differences can be automatically detected, and have a plan in place for how you would fully test a new version of these contracts if they do indeed change unexpectedly. Consider creating a forked version of the file rather than importing it from the package, and manually patch your fork as changes are made.

Functions calling contracts/addresses with transfer hooks are missing reentrancy guards
Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.

Use Ownable2Step rather than Ownable
Ownable2Step and Ownable2StepUpgradeable prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.

block.timestamp is vulnerable to MEV attacks
According to EIP-2612, signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.

tokenURI() does not follow EIP-721
The EIP states that tokenURI() "Throws if _tokenId is not a valid NFT", which the code below does not do. If the NFT has not yet been minted, tokenURI() should revert

Open TODOs
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

Calls to _get() will revert when totalSupply() returns zero
totalSupply() being zero will result in a division by zero, causing the transaction to fail. The function should instead special-case this scenario, and avoid reverting.

latestAnswer() is deprecated
Use latestRoundData() instead so that you can tell whether the answer is stale or not. The latestAnswer() function returns zero if it is unable to fetch data, which may be the case if ChainLink stops supporting this API. The API and its deprecation message no longer even appear on the ChainLink website, so it is dangerous to continue using it.

Use of a single-step ownership transfer
The existing transferOwnership function immediately transfers ownership to the new addres. Consider implementing a two-step variant, where the 'acceptor' of the ownership must call a separate function in order for the transfer to take effect. This can help to prevent mistakes where the wrong address is used, and ownership is irrecoverably lost.

There is one instance of this issue:

File: contracts/NativeTokenFactory.sol

56       function transferOwnership(uint256 tokenId, address newOwner, bool direct, bool renounce) public onlyOwner(tokenId) {
57           if (direct) {
58               // Checks
59               require(newOwner != address(0) || renounce, "NTF: zero address");
60   
61               // Effects
62               emit OwnershipTransferred(tokenId, owner[tokenId], newOwner);
63               owner[tokenId] = newOwner;
64               pendingOwner[tokenId] = address(0);
65           } else {
66               // Effects
67               pendingOwner[tokenId] = newOwner;
68           }
69:      }






















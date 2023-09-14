## [L-01] Vulnerability in ERC20 Contract: Lack of Rejection for Malleable Signatures

## Impact
The impact of this vulnerability is relatively low in the current state of development. However, it is essential to address it to prevent potential risks in the future.

The vulnerability stems from the fact that the ERC20 contract does not reject malleable signatures. Signatures with high S values were restricted in Ethereum Improvement Proposal 2 (EIP-2), specifically in the Homestead upgrade. All transaction signatures with an S-value greater than `secp256k1n/2` are considered invalid. However, the ERC20 contract does not currently enforce this restriction, which could lead to issues in the future.

The specific function affected is `ERC20._isValidSignature`, which does not reject malleable signatures. While this may not pose an immediate risk in the current context, it leaves room for vulnerabilities to be exploited in future upgrades or usages of the contract.

Consider using OpenZeppelin’s ECDSA library which checks for higher S value before calling `ecrecover`: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol

## Instances
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L206

## [L-02] Unused Variable in Root Contract: Escrow

## Impact
This vulnerability also has a minimal impact on the current state of the contract and its functionality. However, it is worth noting that the variable "escrow" in the Root contract is declared but not used anywhere in the contract's code. This unused variable does not affect the contract's behavior or pose any immediate risk.

## Instances
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L35
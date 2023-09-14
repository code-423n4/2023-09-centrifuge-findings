## Findings Summary

| Label | Description |
| - | - |
| [L-01](#l-01-project-may-fail-to-be-deployed-to-chains-not-compatible-with-shanghai-hardfork) | Project may fail to be deployed to chains not compatible with Shanghai hardfork|
| [L-02](#l-02-need-to-check-bytes32-address-for-formatting-correctness) |need to check bytes32 address for formatting correctness|


## [L-01] Project may fail to be deployed to chains not compatible with Shanghai hardfork

All of the contracts in scope have the version pragma fixed to be compiled using Solidity 0.8.21. This [new version of the compiler](https://github.com/ethereum/solidity/releases/tag/v0.8.20) uses the new PUSH0 opcode introduced in the Shanghai hard fork, which is now the default EVM version in the compiler and the one being currently used to compile the project.

Recommendation

Change the Solidity compiler version to 0.8.19 or define an evm version that is compatible across all of the intended chains to be supported by the protocol (see https://book.getfoundry.sh/reference/config/solidity-compiler?highlight=evm_vers#evm_version).


## [L-02] need to check bytes32 address for formatting correctness

Currently the protocol uses bytes32 for address in more than one place.
For example
`PoolManager.transfer()`
```solidity
    function transfer(address currencyAddress, bytes32 recipient, uint128 amount) public {
        uint128 currency = currencyAddressToId[currencyAddress];
        require(currency != 0, "PoolManager/unknown-currency");

        SafeTransferLib.safeTransferFrom(currencyAddress, msg.sender, address(escrow), amount);
        gateway.transfer(currency, msg.sender, recipient, amount);
    }
```



The above `bytes32 recipient` doesn't add the judgment that the last 12 bytes are 0. If the user mistakenly passes in the first 12 bytes as 0, then the user doesn't realize that the parsed address is wrong, resulting in the loss of the token.

It is recommended to check the legitimacy of all `bytes32 addrsss` (the last 12 bytes must be forced to be 0).

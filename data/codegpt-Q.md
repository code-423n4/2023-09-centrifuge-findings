# 1. Symbol not included in the domain separator
In ERC20.sol, the symbol is not included in the domain separator, since such value can be updated through the `file` function, it is recommended to include the `symbol` in the domain separator.
```solidity
    function _calculateDomainSeparator(uint256 chainId) private view returns (bytes32) {
        return keccak256(
            abi.encode(
                keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
                keccak256(bytes(name)),
                keccak256(bytes(version)),
                chainId,
                address(this)
            )
        );
    }
```

# 2. EVM incompatibility with `PUSH0`
Since the Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new `PUSH0` op code. 

This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier [EVM](https://docs.soliditylang.org/en/v0.8.20/using-the-compiler.html?ref=zaryabs.com#setting-the-evm-version-to-target) [version](https://book.getfoundry.sh/reference/config/solidity-compiler#evm_version)

The reason to raise this issue is that, during the discussion with the project team, it seems Centrifuge has not decided which chain to deploy yet.

https://discord.com/channels/810916927919620096/1148615978704453739/1149804641912103033



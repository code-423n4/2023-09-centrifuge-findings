## 1. `DOMAIN_SEPARATOR` is calculated with empty name
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L67-L77

`_DOMAIN_SEPARATOR` is calculated in constructor, and cached until block.chainId is updated. Parameter `name` is used, on the moment of deploy it is empty string. Name is supposed to be later via function `file()`. It introduces inconsistency between real name and name used in domainSeparator
```solidity
    constructor(uint8 decimals_) {
        decimals = decimals_;
        wards[_msgSender()] = 1;
        emit Rely(_msgSender());

        deploymentChainId = block.chainid;
        _DOMAIN_SEPARATOR = _calculateDomainSeparator(block.chainid);
    }

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
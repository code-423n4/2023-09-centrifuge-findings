## Finding: Potential Interface Mismatch in Critical Contract Updates

**Severity**: Low/QA

### Description:

Several contracts in the system provide functionality to update critical components or addresses. These include `InvestmentManager`, `LiquidityPool`, `Gateway`, `PoolManager`, `Trancher.sol`, and modifications to `ERC20`. The current implementations do not consistently ensure that the updated addresses or contracts adhere to the expected interfaces. If an incorrect address or contract is set, it could lead to unexpected behavior or function failures.

### Recommendation:

For all the mentioned contracts, integrate the ERC-165 standard checks to ensure that any new address or contract set supports the expected interfaces before they're applied.

**Example Updated Function with ERC-165 checks (from `InvestmentManager`)**:

```solidity
function file(bytes32 what, address data) external auth {
    IERC165 target = IERC165(data); // Typecast to ERC-165 interface
    if (what == "gateway") {
        require(target.supportsInterface(type(GatewayLike).interfaceId), "Contract does not implement GatewayLike");
        gateway = GatewayLike(data);
    } 
    else if (what == "poolManager") {
        require(target.supportsInterface(type(PoolManagerLike).interfaceId), "Contract does not implement PoolManagerLike");
        poolManager = PoolManagerLike(data);
    } 
    else revert("InvestmentManager/file-unrecognized-param");

    emit File(what, data);
}

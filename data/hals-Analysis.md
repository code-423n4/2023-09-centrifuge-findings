## Audit Scope

Centrifuge operates on a hub-and-spoke model, managing RWA pools on its dedicated Centrifuge Chain and deploying Liquidity Pools on other networks. This facilitates efficient communication and expands accessibility.
The audited codebase encompasses approximately 2657 non-commented source lines of code (nSLoC), distributed across 18 contracts.

## Approach Taken

Audit approach encompassed several key steps:

1. **Comprehensive Understanding:** Began by gaining a profound comprehension of the protocol's objectives and architectural blueprint through the provided documentation.
2. **Manual Code Review:** A detailed manual code review followed, aimed at uncovering potential vulnerabilities and identifying instances where the code diverges from its intended design, thus denoting potential bugs.
3. **Known Vulnerabilities Match:** Scrutinized the code for patterns resembling known vulnerabilities reported in similar projects.
4. **Documentation:** Documented findings.

## Architecture Feedback

- **Readability Enhancement:** The codebase could be significantly improved in terms of readability. There's noticeable code repetition when interacting with other contracts through interfaces. It is recommended to create a dedicated folder for project interfaces, making them accessible wherever contract interactions via interfaces occur.
- **Authorization Logic:** The `Root` contract inherits some authorization logic from `Auth`, but this logic is also repeated within the contract. Consolidating this logic would streamline contract maintenance.

## Centralization Risks

The `Root` contract holds the authoritative position as the ward (authorized entity) over all project contracts. The `Root` contract's wards can grant or revoke permissions (add or remove wards) on any contract within the system. This setup presents centralization risks, as every ward wields control over the addition and removal of wards across the entire system.

## Systemic Risks

- **Overall Codebase Strength:** The codebase demonstrates robustness and adheres to security best practices.
- **Identified Vulnerabilities:** While the project is solid, a few vulnerabilities were detected:
  1. **LiquidityPool Contract:** Malicious users could exploit the liquidity pool share token balance using low-level calls implemented in certain functions (`transfer`, `transferFrom`, and `approve`).
  2. **Root Contract:** Wards in the `Root` contract can circumvent the timelock requirement for adding wards by directly calling the `Root.rely` function, which doesn't enforce the execution delay.
  3. **Missing Pause Checks:** The `Root` contract's functions lack checks to verify if the contract is paused before executing `executeScheduledRely`.
  4. **Inaccessible Root Contract:** There's a risk of rendering the `Root` contract inaccessible if the last contract ward removes themselves, effectively denying any further access.
  5. **ERC20 Functions:** The `ERC20.transfer` and `ERC20.transferFrom` functions may result in silent balance overflows during the receiver's update.

## Other Recommendations

1. **`Root` Contract Enhancements:** Implement a wards counter in the `Root` contract, which can be checked when wards are removed. This ensures that the contract doesn't become inaccessible when all wards are removed.
2. **Disabling `Root.rely`:** Disable the inherited `Root.rely` function to enforce that adding wards follows the timelock delay instead of immediate addition.
3. **Wards Restriction:** Consider limiting the ability to add wards to a single externally owned account (EOA), potentially the admin of the `Root` contract.

## Time Spent

The audit process consumed approximately 24 hours, divided between manual code review, thorough analysis of the provided documentation, and documenting my findings.


### Time spent:
24 hours
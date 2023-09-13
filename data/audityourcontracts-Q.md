## [01] No support for ERC-2098 compact signatures
In it's current form Centrifuge's permit functionality would not support [ERC-2098](https://eips.ethereum.org/EIPS/eip-2098) compact signatures.

## Recommendation
This seems like functionality that Centrifuge may want to add support for [ERC-2098](https://eips.ethereum.org/EIPS/eip-2098). For example popular protocols like OpenSea's Seaport [supports it](https://github.com/ProjectOpenSea/seaport/blob/539f0c18af85152aff9d64d90a55cf1627fd3e25/reference/lib/ReferenceSignatureVerification.sol#L59).
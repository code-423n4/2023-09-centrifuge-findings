# [L -01] It is possible to change the name/symbol of the tranche-token after the creation

## Affected contract/function:

`Tranche.sol` => `TrancheTokenLike.file()`

## Description:

There is no control over the name/symbol of the token once the tranche token has been created and they can be changed. Although there is no direct way to still funds, this could lead to a malicious creator perpetrating some form of fraud on investors.

## Mitigation:

Store the values after the creation of the tranche token.

# [L -02] increaseAllowance and decreaseAllowance are dangerous and no longer ERC20 standard

## Affected contract/function:

`Tranche.sol` => `ERC.sol/increaseAllowance()` / `ERC.sol/decreaseAllowance()`

## Description:

Recently, Open Zeppelin removed these functions from the ERC.sol standard because they were being used for fraud and phising (as they are external).

## Mitigation:

Use safeErc20.sol, where those functions are internals, or only the IERC.sol where those functions doesn´t exists.
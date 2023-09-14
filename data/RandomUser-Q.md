Contract : PoolManager
Function : allowPoolCurrency()

It is possible to allow already allowed currency. Though it doesn't affect the system it may be misleading

Consider adding a condition that checks it

// ============================================================================

Contract : PoolManager
Function : updateTrancheTokenMetadata()

Function file() in TrancheToken doesn't handle "name" and "symbol" arguments. So the entire point of the function PoolManager::updateTrancheTokenMetadata() is missing.

Consider adding conditions in TranceToken::file() that handle agruments "symbol" and "name"
 
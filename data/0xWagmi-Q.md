# [L-01]  TrancheTokens can be transferred to zero address in centrifuge chain  

###  Impact
In `PoolManager.sol` it has a function `transferTrancheTokensToCentrifuge` which  takes `destinationAddress` as params in `bytes32`
which is an address in centrifuge chain assuming if centrifuge chain use bytes32(0x0) as the 0 address, A user can transfer  Tranche Tokens
from EVM chain to centrifuge chain which could result in loss of tokens .

### POC
```diff
  function testTransferTrancheTokensToCentrifuge(
        uint64 validUntil,
        bytes32 centChainAddress,
        uint128 amount,
        uint64 poolId,
        uint8 decimals,
        string memory tokenName,
        string memory tokenSymbol,
        bytes16 trancheId,
        uint128 currency
    ) public {
        vm.assume(decimals <= 18);
        vm.assume(currency > 0);
        vm.assume(validUntil > block.timestamp + 7 days);
+        centChainAddress = bytes32(0x0);
        address lPool_ = deployLiquidityPool(poolId, decimals, tokenName, tokenSymbol, trancheId, currency);
        homePools.updateMember(poolId, trancheId, address(this), validUntil);

        // fund this account with amount
        homePools.incomingTransferTrancheTokens(poolId, trancheId, uint64(block.chainid), address(this), amount);

        // Verify the address(this) has the expected amount
        assertEq(LiquidityPool(lPool_).balanceOf(address(this)), amount);

        // Now send the transfer from EVM -> Cent Chain
        LiquidityPool(lPool_).approve(address(poolManager), amount);
        poolManager.transferTrancheTokensToCentrifuge(poolId, trancheId, centChainAddress, amount);
        assertEq(LiquidityPool(lPool_).balanceOf(address(this)), 0);

        // Finally, verify the connector called `router.send`
        bytes memory message = Messages.formatTransferTrancheTokens(
            poolId,
            trancheId,
            bytes32(bytes20(address(this))),
            Messages.formatDomain(Messages.Domain.Centrifuge),
            centChainAddress,
            amount
        );
        assertEq(mockXcmRouter.sentMessages(message), true);
    }

```

### Mitigation
consider reverting  if the token are  sending to  the 0 address on the centrifuge chain  

 ## [N-1]Delegatecalling Precompiles on centrifuge chain result in providing privilege access to caller.
 
 Vulnerabilities in precompiles are the one of most interesting  and rewarding vulnerabilities found in 2022, Even though this is out of scope  when I was starting out  caught  my eye  on `https://github.com/centrifuge/centrifuge-chain/blob/main/pallets/liquidity-pools-gateway/axelar-gateway-precompile/src/lib.rs` ,I just quickly checked  [PwningETH 1 Million Bounty](https://pwning.mirror.xyz/okyEG4lahAuR81IMabYL5aUdvAsZ8cRCbYBXh8RHFuE) blog post to get some more context, this is pretty much similar to `moonbeam/precompiles/balances-erc20/src/lib.rs`      

 *vulnerability in moonbeam*  

```rust 
fn execute(&self, handle: &mut impl PrecompileHandle) -> Option<PrecompileResult> {
  match handle.code_address() {
    // Ethereum precompiles :
    a if a == hash(1) => Some(ECRecover::execute(handle)),
    a if a == hash(2) => Some(Sha256::execute(handle)),
    a if a == hash(3) => Some(Ripemd160::execute(handle)),
    a if a == hash(5) => Some(Modexp::execute(handle)),
  ...
  }
  }
```

the dev fixed the issue by adding, 

```diff
fn execute(&self, handle: &mut impl PrecompileHandle) -> Option<PrecompileResult> {
  // Filter known precompile addresses except Ethereum officials
+  if self.is_precompile(handle.code_address())
+   && handle.code_address() > hash(9)
+   && handle.code_address() != handle.context().address
+ {
+   return Some(Err(revert(
+     "cannot be called with DELEGATECALL or CALLCODE",
+   )));
+  }

    match handle.code_address() {
......

```

When  compared  it  with   `https://github.com/centrifuge/centrifuge-chain/blob/main/pallets/liquidity-pools-gateway/axelar-gateway-precompile/src/lib.rs#L247` in  `Upgrade 1020 (#1522)` (latest patch) and it seems like the exactly the same vulnerability, I asked the sponsor about this and he  send me a link  to a PR related to this and  haven't merged yet,having some  merge conflicts  ,  `https://github.com/centrifuge/centrifuge-chain/pull/1538/files#diff-6ff71e8cbdaf7388017ce533cdeba3383ff6052b1d52d73f36b75b652a4e7fc6R256`. So it means  the team is aware about it and still working on it. I hope the things will be fixed soon ,I think it's better consider getting  a security review on precompiles        

[N-02] Usage of mock contracts instead of actual contracts 

It is good to test the this as realistic  as possible on mainnet fork rather than using mocks with less access control .. 








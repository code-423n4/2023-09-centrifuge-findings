# QA report

## **Table of Contents**

| |Issue|
|-|:-|
| QA&#x2011;01 | Removing a pool that's already being removed does not revert |
| QA&#x2011;02 | `_msgSender()` used without implementing the `isTrustedForwarder()` function of EIP-2771 |
| QA&#x2011;03 | The contracts can't access the LiquidityPool.sol contract via permits |
| QA&#x2011;04 | Increase/decreaseAllowance() are now removed from OZ's ERC20 implementation due to a valid argument and same should be done for Centrifuge's |
| QA&#x2011;05 | Deadlines just like stale checks shouldn't be inclusive |
| QA&#x2011;06 | Context.sol does not implement `_msg.data()` which leads to nuances when dealing with meta transactions |
| QA&#x2011;07 | Add a mechanism to remove a currency |
| QA&#x2011;08 | Refactor the check in `executeScheduledRely()` |
| QA&#x2011;09 | Authorizations are not given to necessary admins |

## QA-01 Removing a pool that's already being removed does not revert

### Impact

Low, a flaw in contract's actions

> NB: Same thing can be said for `addLiquidity()`, i.e adding an already set pool does not revert
> Key to note the `addIncomingRouter()` and `removeIncomingRouter()` from Gateway.sol have a similar flaw

### Vulnerability Details

Take a look at `addLiquidityPool()`

```solidity
    function addLiquidityPool(address liquidityPool) public auth {
        liquidityPools[liquidityPool] = true;
        emit AddLiquidityPool(liquidityPool);
    }

    function removeLiquidityPool(address liquidityPool) public auth {
        liquidityPools[liquidityPool] = false;
        emit RemoveLiquidityPool(liquidityPool);
    }
```

As seen, both setter functions do not actually check if their state is the state they'd like to change the respective pool to, this can be proven using the modified test case from `Tranche.t.sol`as POC, add this to `Tranche.t.sol` and it runs without failing

```solidity
    function testRemoveLiquidityPool() public {
        token.addLiquidityPool(self);
        assertTrue(token.isTrustedForwarder(self));

        // success
        token.removeLiquidityPool(self);
        token.removeLiquidityPool(self);
        token.removeLiquidityPool(self);
        token.removeLiquidityPool(self);
        token.removeLiquidityPool(self);
        token.removeLiquidityPool(self);
        assertTrue(!token.isTrustedForwarder(self));

        // remove self from wards
        token.deny(self);
        // auth fail
        vm.expectRevert(bytes("Auth/not-authorized"));
        token.removeLiquidityPool(self);
    }
```

As seen five additional calls to `removeLiquidity()` are made with none failing

### Recommendation

Setter functions generally should have equaliser checkers to ensure tha hey are not attempted to be set to their previously set value... this should also be integrated

## QA-02 `_msgSender()` used without implementing the `isTrustedForwarder()` function of EIP-2771

### Impact

Low... Protocol does not correctly follow the Eip's specification

### Vulnerability Details

Take a look at [Context.sol#L4-L15](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Context.sol#L4-L15)

<details>

<summary>Click to see full affected code reference</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

/// @title  ERC1271 Context
/// @dev    Provides information about the current execution context, including the
///         sender of the transaction and its data. While these are generally available
///         via msg.sender and msg.data, they should not be accessed in such a direct
///         manner, since when dealing with meta-transactions the account sending and
///         paying for execution may not be the actual sender (as far as an application
///         is concerned).
/// @dev    Adapted from OpenZeppelin Contracts v4.4.1 (utils/Context.sol)
abstract contract Context {
    function _msgSender() internal view virtual returns (address) {
        return msg.sender;
    }
}
```

</details>

Now would be key to note that [EIP-2771](https://eips.ethereum.org/EIPS/eip-2771) states that "To provide this discovery mechanism a Recipient contract MUST implement this function:" and then outlines the requirements of the `isTrustedForwarder()` function. This contract does not provide such a function, so the contract does not properly follow the specification, and therefore this is a low-severity issue.

### Recommendation

If trusted forwarders are meant to be used, the contract should properly extend ERC 2771

## QA-03 The contracts can't access the LiquidityPool.sol contract via permits

### Impact

Sits on the fence of medium & low, since this means that some users can't access an import part of the protocol... crux of the issue is that only one `permit()` function is being specified in the `ERC20PermitLike` interface in LiquidityPool.sol where as two of the permit() functions exist with one accepting signatures via v,r,s and the other using ERC1271 to support contract signed signatures and it's the one with the

### Vulnerability Details

See summary

Now take a look at [LiquidityPool.sol#L10-L14](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/LiquidityPool.sol#L10-L14)

```solidity
interface ERC20PermitLike {
    function permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
        external;
    function PERMIT_TYPEHASH() external view returns (bytes32);
    function DOMAIN_SEPARATOR() external view returns (bytes32);
}

```

As seen only permits passed using v,r,s are allowed which means contracts are not allowed since EIP 1271 is not provided to validate the signatures provided by contracts

### Recommendation

Implement both functions or specifically document that only EOA is to access this part of the protocol

## QA-04 Increase/decreaseallowance() are now removed from OZ's ERC20 implementation due to a valid argument and same should be done for Centrifuge's

### Impact

Do note that whereas these functions are not part of the [EIP-20](https://eips.ethereum.org/EIPS/eip-20) specs.
These functions may allow for further phishing possibilities (instead of the common approve or permit ones; see e.g. just last week someone lost $24m since he got tricked into signing a malicious increaseAllowance payload https://etherscan.io/tx/0xcbe7b32e62c7d931a28f747bba3a0afa7da95169fcf380ac2f7d54f3a2f77913).

### Vulnerability Details

Take a look at [ERC20.sol#L139-L159](https://github.com/code-423n4/2023-09-centrifuge/blame/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L139-L159)

```solidity
    function increaseAllowance(address spender, uint256 addedValue) external returns (bool) {
        uint256 newValue = allowance[_msgSender()][spender] + addedValue;
        allowance[_msgSender()][spender] = newValue;

        emit Approval(_msgSender(), spender, newValue);

        return true;
    }

    function decreaseAllowance(address spender, uint256 subtractedValue) external returns (bool) {
        uint256 allowed = allowance[_msgSender()][spender];
        require(allowed >= subtractedValue, "ERC20/insufficient-allowance");
        unchecked {
            allowed = allowed - subtractedValue;
        }
        allowance[_msgSender()][spender] = allowed;

        emit Approval(_msgSender(), spender, allowed);

        return true;
    }
```

The security concerns that fix `increaseAllowance()` and `decreaseAllowance()` are not critical nor high _decreaseAllowance can be frontrunned also_ and thus the implementation of this should be rethought.

### Recommendation

Normal approvals should work just fine and should be used instead

## QA-05 Deadlines just like stale checks shouldn't be inclusive

### Impact

Medium to low, since this is only for the instance of `block.timestamp` which could manipulated by an attacker for up to ~900 seconds

NB: Report only covers the instance with the `permit()` from ERC20.sol but the same window is applicable to restriction manager's `validUnitl`, as seen [here]() members are still allowed access even after their duration... _do note that this is even **more** severe than the issue with permit, since this can be seen as somewhat of a side step of users `membership validity` if they can extend it for an extreme of ~ 900 seconds_

### Vulnerability Details

Take a look at the [ERC20.sol#L216-L237](https://github.com/code-423n4/2023-09-centrifuge/blame/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L216-L237)

```solidity
    function permit(address owner, address spender, uint256 value, uint256 deadline, bytes memory signature) public {
        require(block.timestamp <= deadline, "ERC20/permit-expired");
        require(owner != address(0), "ERC20/invalid-owner");

        uint256 nonce;
        unchecked {
            nonce = nonces[owner]++;
        }

        bytes32 digest = keccak256(
            abi.encodePacked(
                "\x19\x01",
                block.chainid == deploymentChainId ? _DOMAIN_SEPARATOR : _calculateDomainSeparator(block.chainid),
                keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonce, deadline))
            )
        );

        require(_isValidSignature(owner, digest, signature), "ERC20/invalid-permit");

        allowance[owner][spender] = value;
        emit Approval(owner, spender, value);
    }

```

As seen the check for the expiration of signature is inclusive and if `deadline == block.timestamp` the execution still goes through, the issue with this approach is that a user might make a valid assumption that it's 12s per block and depending on the context on user's end they might have approved a different personnel a specific time that they can be able to spend their until, but this is not the case as the personnel till has access even after that specific time, since it's inclusive, with the difference being 12 seconds and as high as 900 seconds in manipulative instances

<details>

<summary>Click to see a short POC proving this</summary>
   function testPermit() public {
        uint256 privateKey = 0xBOB;
        address owner = vm.addr(privateKey);

        (uint8 v, bytes32 r, bytes32 s) = vm.sign(
            privateKey,
            keccak256(
                abi.encodePacked(
                    "\x19\x01",
                    token.DOMAIN_SEPARATOR(),
                    keccak256(abi.encode(PERMIT_TYPEHASH, owner, address(0xALICE), 1e18, 0, block.timestamp))
                )
            )
        );

        vm.expectEmit(true, true, true, true);
        emit Approval(owner, address(0xALICE), 1e18);
        token.permit(owner, address(0xALICE), 1e18, block.timestamp, v, r, s);

        assertEq(token.allowance(owner, address(0xALICE)), 1e18);
        assertEq(token.nonces(owner), 1);
    }

</details>

### Recommendation

Deadlines shouldn't be inclusive, and if code is to be kept as is, users should be informed that their approvals could still go through a little while after their specified deadlines

## QA-06 Context.sol does not implement `_msg.data()` which leads to nuances when dealing with meta transactions

### Impact

Low... In the case where msg.data is going to be used, provision of the current execution's context might encounter a few problem, since msg.data or msg.sender shouldn't be accessed in a direct manner

### Vulnerability Details

Take a look at [Context.sol#L4-L15](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/util/Context.sol#L4-L15)


<details>

<summary>Click to see full affected code reference</summary>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

/// @title  ERC1271 Context
/// @dev    Provides information about the current execution context, including the
///         sender of the transaction and its data. While these are generally available
///         via msg.sender and msg.data, they should not be accessed in such a direct
///         manner, since when dealing with meta-transactions the account sending and
///         paying for execution may not be the actual sender (as far as an application
///         is concerned).
/// @dev    Adapted from OpenZeppelin Contracts v4.4.1 (utils/Context.sol)
abstract contract Context {
    function _msgSender() internal view virtual returns (address) {
        return msg.sender;
    }
}
```

</details>

As seen clearly noted by and referenced, nuances could be encountered when dealing with meta transactions and why msg.sender and msg.data shouldn't be accessed in a direct manner, but the context.sol contract currently only includes the helper to get the msg.seder when dealing with meta transactions and not the the msg.data

### Recommendation

Add a helper to get the msg.data, i,e

```solidity
    function _msgData() internal view virtual returns (bytes calldata) {
        return msg.data;
    }
```

This can also be seen to be followed in adopted Openzeppelin contract's [Context.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/36bf1e46fa811f0f07d38eb9cfbc69a955f300ce/contracts/utils/Context.sol#L21-L23) from

## QA-07 Add a mechanism to remove a currency

### Impact

Low, info... in the case where Centrifuge no longer wants to receive information about a currency they can't be removed

### Vulnerability Details

### Recommendation

Implement something like a `removeCurrency()` function that allows for the removal of a currency, with proper checks and balances. Ensure that this is only callable by authorized addresses, and that there are safety mechanisms in place to prevent potential loss of funds.

## QA-08 Refactor the check in `executeScheduledRely()`

### Impact

Low... the requirement of `schedule[target] < block.timestamp` leads to a stall of executing scheduled rely for a short period of time even when it's time to execute the schedural

### Proof of Concept

The below steps have been outlined as the steps that'd be taken in the case where someone gains control over a router and signs a malicious message, quoting the [docs](https://github.com/code-423n4/2023-09-centrifuge/tree/main#access-setup):

Someone gains control over a router and triggers a malicious ScheduleUpgrade message

> Within 48h, the pause admin calls `pause() `to block further incoming messages from the router. The delayed admin calls `cancelRely()` to cancel the scheduled rely from the router exploiter.
> The delayed admin submits a `scheduleRely()` to remove the router from the gateway contract 48h later.
> After 48h, the router is removed and cannot interact with the system anymore.

But the above can be seen not to be implemented in code, take a look at [Root.sol#L75-L83](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/Root.sol#L75-L83)

```solidity
   function executeScheduledRely(address target) public {
        require(schedule[target] != 0, "Root/target-not-scheduled");
        //@audit
        require(schedule[target] < block.timestamp, "Root/target-not-ready");

        wards[target] = 1;
        emit Rely(target);

        schedule[target] = 0;
    }

```

As seen, while executing a scheduled rely, the check on the time is having a `<` instead of `<=` which means that unlike how it's explained in the docs, after `DELAY` has passed which is initially set to 48 hours the scheduled executions wouldn't be immeditaely accessible and can even be stalled for _~900 seconds_ if a miner decides to be malicious

### Recommendation

Refactor ` executeScheduledRely()` to make the right checks as hinted in _Proof Of Concept_

## QA-09 Authorizations are not given to necessary admins

### Impact

Low... since the admin can still access these functions, but this can be specified as a break of contract's logic since natspec comments have specified these functions to be either accessible by owner or authorised admins

### Vulnerability Details

Take a look at the [LiquidityPool.sol]() contract, all functions prefixed with the `withApproval(owner)` modifier have claims that, _@notice Request can only be called by the Owner of the assets or an authorized admin._

But this is not possible since the below is the current implementation of the `withApproval()` modifier

```solidity
    modifier withApproval(address owner) {
        require(msg.sender == owner, "LiquidityPool/no-approval");
        _;
    }
```

### Recommendation

Authorize necessary admins and allow them access to functions or clearly state that only owners can access these functions

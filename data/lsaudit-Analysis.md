During the security review phase, my main focus had been directed at the protocol's access control. Below sections of the analysis mostly describe issues related to its implementation.

The documentation provided by the Sponsors very comprehensively describes the authorization mechanism in the `Access control` paragraph. I do admire the fact, that so much documentation space has been dedicated to describe the Access Control mechanism, thus I had decided that my main focus during the security review - will be directed at evaluating every pros and cons of that mechanism and identify any potential issue related to it.

# Approach taken

The first step of the security review was getting familiar with provided documentation. Fortunately, provided docs describes the topic of Access Control very extensively.
However, the only inconsistency, I found during the documentation review process, is the below sentence on the contest's page: `Authentication uses the ward pattern, in which addresses can be relied or denied to get access`. Actually, in the context of the ward pattern – we are dealing with authorization, not the authentication, since wards are not authenticated, but just authorized to execute `rely()` an `deny()` methods.

I personally enjoyed the fact, that documentation describes multiple of emergency scenarios (e.g. someone triggers malicious `scheduleUpgrade` or `pause()` function). Having those emergency scenarios described and developing the protocol by keeping in mind that those emergency scenarios may occur – implies, how seriously the security of the protocol is treated.

After getting familiar with the documentation, I started to evaluate a security of the proposed mechanism, keeping my focus mostly on the files which implement or use the access control mechanism.

I evaluated the risks and threats (described in the `Access Control Analysis Conclusion` paragraph) related to Access Control mechanism. I also verified if described emergency scenarios are indeed working as described in the docs.

Another part of the code-base I focused on was the Restriction Manager (`RestrictionManager.sol`, `Tranche.sol`). Since documentation describes it as `ERC1404 based contract that checks transfer restrictions` - I got familiar with EIP-1404 and I verified if it is compliant with this standard. Identified risks and threats are described in `Restriction Manager Analysis Conclusion`.

# Access Control Analysis Conclusion

The protocol solution for most of the described emergency scenarios lies in the time lock implementation. Nobody should be able to become a ward on any contract before the time lock. Based on that assumption – it is extremely crucial to make sure that the time lock mechanism is implemented properly.
For many investors, the protocol's security and risk-mitigation mechanism are the main factor which allows them to decide if they want to invest their funds in the protocol. Having properly implemented time lock is indeed a relevant factor.
However, the time lock mechanism can be fully disabled in `Root.sol`. While the time lock max length is defined to 4 weeks, it turns out that there is no lower limit when it comes to its duration. It can be set to 0 (or 1 second), which is basically equivalent to turning it off. Taking into consideration how important for security the time lock is – it should not be possible to disable it under any circumstances.


Another important issue is related to the `ward` pattern. Every contract should have at least one `ward`. Without any `ward` contract won't be functioning properly – since most of the methods are callable only by ward.  None of the contracts enforce the rule of having at least one `ward`. Any `ward` can accidentally call `deny()` method on their own address (or accidentally pass their own address to `deny()`) and remove themself from being a `ward`. When the last `ward` removes themself from being a `ward` - protocol would become ward-less and broken. 
There should be additional mechanism implemented in the `deny()` method which guarantees, that `ward` cannot remove themself (e.g. passing `msg.sender` to that method should revert).

Every emergency scenarios described in the docs protect from adding malicious ward. There is, however, no emergency scenario when already added `ward` becomes malicious. It's crucial to notice, that malicious ward can actually do everything within the contract. This is, however, unavoidable. According to that – some centralization risk occurs. Adding a single EOA as `ward` is a centralization risk and a single point of failure. A single private key may be taken in a hack, or the sole holder of the key may become unable to retrieve the key when necessary. Moreover, the more wards are added, the higher chances are that one of them might become malicious.
This risk should be evaluated every time the new `ward` is being added.


# Restriction Manager Analysis Conclusion

As EIP-1404 standard requires, function `detectTransferRestriction` must be evaluated inside token's `transfer` and `transferFrom`. The logic of this function is left to the issuer.
The developers decided to use this function to verify if the address is on the memberlist when receiving the token.
The members are being added (or removed) via `updateMember()` function. I think that the better approach would be creating two additional functions: `addMember()` and `removeMember()`, instead of using `updateMember()` to both adding and/or removing members.
The current implementation does not allow to immediately remove the member. They can only be removed in some point in the future:

```
    function updateMember(address user, uint256 validUntil) public auth {
        require(block.timestamp <= validUntil, "RestrictionManager/invalid-valid-until");
        members[user] = validUntil;
    }
```

If `updateMember()` (called with `validUntil` close to `block.timestamp`) stuck in the mem-pool and would be mined when `validUntil` > `block.timestamp` - `updateMember` would revert and user wouldn't be removed from the memberlist.
Having a function which immediately removes the member from the memberlist is more secure approach.

# Code base quality

As previously mentioned – this analysis mainly focuses on access control and Restriction Manager – thus the code-base quality is also evaluated within that context.

Most of the functions do not implement any NatSpec. Comments are also confusing.
For example, the NatSpec of `DelayedAdmin.sol` is copied-pasted into `PauseAdmin.sol` (`PauseAdmin` contract is described as `@title  Delayed Admin`), which obviously is not true.

The `RestrictionManager.sol`, even described as `ERC1404 based contract` - does not provide any explanation – why exactly it is implemented as ERC1404. By diving deeper into the code-base, this becomes clear – however, the code-base should include some comments, which will help to understand the overall purpose of the code, before actually reading it.



## Time spent
12h (including 8h for reviewing the access control and Restriction Manager mechanisms).

### Time spent:
12 hours
# Centrifuge: Analysis
Audit contest: `2023-09-centrifuge`

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. Code Audit Approach](#2-code-audit-approach)
  - [2.1 Audit Documentation and Scope](#21-audit-documentation-and-scope)
  - [2.2 Setup and Tests](#22-setup-and-tests)
  - [2.3 Code review](#23-code-review)
  - [2.4 Threat Modelling](#24-threat-modelling)
  - [2.5 Exploitation and Proofs of Concept](#25-exploitation-and-proofs-of-concept)
  - [2.6 Report Issues](#26-report-issues)
- [3. Architecture overview](#3-architecture-overview)
  - [3.1 `Ward` Pattern](#31-ward-pattern)
    - [3.1.1 The `PauseAdmin`](#311-the-pauseadmin)
    - [3.1.2 The `DelayedAdmin`](#312-the-delayedadmin)
    - [3.1.3 A Problem with `DelayedAdmin`](#313-a-problem-with-delayedadmin)
      - [3.1.3.1 The Delayed Rescue](#3131-the-delayed-rescue)
      - [3.1.3.2 The Prompt Rescue](#3132-the-prompt-rescue)
  - [3.2 The Spell Pattern](#32-the-spell-pattern)
  - [3.3 Pausing the Protocol](#33-pausing-the-protocol)
    - [3.3.1 Redundancy?](#331-redundancy)
  - [3.4 Liquidity Pools](#34-liquidity-pools)
  - [3.5 Upgrading the Protocol](#35-upgrading-the-protocol)
  - [3.6 Centralisation Risks](#36-centralisation-risks)
- [4. Implementation Notes](#4-implementation-notes)
  - [4.1 General Impressions](#41-general-impressions)
  - [4.2 Composition over Inheritance](#42-composition-over-inheritance)
  - [4.3 Tests](#43-tests)
  - [4.4 Comments](#44-comments)
  - [4.5 Solidity Versions](#45-solidity-versions)
- [5. Conclusion](#5-conclusion)

## 1. Executive Summary

For this analysis, I will center my attention on the scope of the ongoing audit contest `2023-09-centrifuge`. I begin by outlining the code audit
approach employed for the in-scope contracts, followed by sharing my perspective on the architecture, and concluding with observations about the
implementation's code.

**Please note** that unless explicitly stated otherwise, any architectural risks or implementation issues mentioned in this document are not to
be considered vulnerabilities or recommendations for altering the architecture or code solely based on this analysis. As an auditor, I acknowledge the
importance of thorough evaluation for design decisions in a complex project, taking associated risks into consideration as one single part of an
overarching process. It is also important to recognise that the project team may have already assessed these risks and determined the most suitable
approach to address or coexist with them.

## 2. Code Audit Approach

Time spent: 18 hours

### 2.1 Audit Documentation and Scope
The initial step involved examining [audit documentation and scope](https://github.com/code-423n4/2023-09-centrifuge)
to grasp the audit's concepts and boundaries, and prioritise my efforts. It is worth highlighting the good
quality of the `README` for this audit contest, as it provides valuable insights and actionable guidance that greatly facilitate
the onboarding process for auditors.

### 2.2 Setup and Tests
Setting up to execute `forge test` was remarkably effortless, greatly enhancing the efficiency of the auditing process.
With a fully functional test harness at our disposal, we not only accelerate the testing of intricate concepts and potential
vulnerabilities but also gain insights into the developer's expectations regarding the implementation. Moreover, we can deliver
more value to the project by incorporating our proofs of concept into its tests wherever feasible.

### 2.3 Code review

The code review commenced with understanding the "`ward` pattern" used to manage authorization accross the system.
Thoroughly understanding this pattern made understanding the protocol contracts and its relations much smoother.
Throughout this stage, I documented observations and raised questions concerning potential exploits without going too deep.

### 2.4 Threat Modelling

I began formulating specific assumptions that, if compromised, could pose security risks to the system. This process aids me in
determining the most effective exploitation strategies. While not a comprehensive threat modeling exercise, it shares numerous similarities with one.

### 2.5 Exploitation and Proofs of Concept

From this step forward, the main process became a loop conditionally encompassing each of the steps 2.3, 2.4, and 2.5, involving attempts at exploitation and
the creation of proofs of concept with a little help of documentation or the ever helpful sponsors on Discord. My primary objective in this phase is to challenge
critical assumptions, generate new ones in the process, and enhance this process by leveraging coded proofs of concept to expedite the development of successful exploits.

### 2.6 Report Issues

Alghough this step may seem self-explanatory, it comes with certain nuances. Rushing to report vulnerabilities immediately and then neglecting them is unwise. The most effective
approach to bring more value to sponsors (and, hopefully, to auditors) is to document what can be gained by exploiting each vulnerability. This assessment helps in evaluating whether
these exploits can be strategically combined to create a more significant impact on the system's security. In some cases, seemingly minor and moderate issues can compound to form a
critical vulnerability when leveraged wisely. This has to be balanced with any risks that users may face. In the context of Code4rena audit contests, zero-days or highly sensitive bugs
affecting deployed contracts are given a more cautious and immediate reporting channel.

## 3. Architecture overview

## 3.1 `Ward` Pattern

In grasping the audited architecture, one must comprehend some simple yet potent principles. The first concept is the `ward`` pattern, employed to oversee
authorisation throughout the protocol. I consider it an elegant and secure method. Below is the 'ward' graph supplied by the Centrifuge team:

![Wards Overview](https://raw.githubusercontent.com/code-423n4/2023-09-centrifuge/main/images/wards.png?raw=true)

And, to better contextualise, this is a quote from the audit's `README`:

```
The Root contract is a ward on all other contracts. The PauseAdmin can instantaneously pause the protocol. The DelayedAdmin can make itself ward on any
contract through Root.relyContract, but this needs to go through the timelock specified in Root.delay. The Root.delay will initially be set to 48 hours.
```

So how does this work in practice?

As `Root` is a ward on all other contracts, any ward on `Root` can use its `relyContract` function to become a ward in any of these contracts:

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L90-L93:
```c++
    function relyContract(address target, address user) public auth {
        AuthLike(target).rely(user);
        emit RelyContract(target, user);
    }
```

The opposite being the `denyContract` function:

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Root.sol#L98-L102:
```c++  
    function denyContract(address target, address user) public auth {
        AuthLike(target).deny(user);
        emit DenyContract(target, user);
    }
}
```

The term "rely" is employed to signify "becoming a ward." Thus, when I state that contract X relies on contract Y, it implies
that contract Y assumes the role of a ward within contract X. Conversely, the verb "deny" is utilised to describe the opposing action.

Contracts within the system are anticipated to inherit (or implement) the [`Auth` contract](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Auth.sol).
The `Auth` contract is exceptionally minimal, and for your reference, I have included it below:

```c++
// SPDX-License-Identifier: AGPL-3.0-only
// Copyright (C) Centrifuge 2020, based on MakerDAO dss https://github.com/makerdao/dss
pragma solidity 0.8.21;

/// @title  Auth
/// @notice Simple authentication pattern
contract Auth {
    mapping(address => uint256) public wards;

    event Rely(address indexed user);
    event Deny(address indexed user);

    /// @dev Give permissions to the user
    function rely(address user) external auth {
        wards[user] = 1;
        emit Rely(user);
    }

    /// @dev Remove permissions from the user
    function deny(address user) external auth {
        wards[user] = 0;
        emit Deny(user);
    }

    /// @dev Check if the msg.sender has permissions
    modifier auth() {
        require(wards[msg.sender] == 1, "Auth/not-authorized");
        _;
    }
}
```

With these capabilities at their disposal, any `ward` in `Root` has the ability to invoke `root.allowContract(address target, address user)`.
This action designates `user` — whether an Externally Owned Account (EOA) or Contract — as a `ward` within the `contract`. As a result,
the `Root`'s existing `wards` can execute functions within any of the contracts to which `Root` serves as a `ward` (i.e. all other contracts
in the system).

### 3.1.1 The `PauseAdmin`

The `PauseAdmin` represents a concise contract equipped with functions to manage `pausers`, those who can call its `pause` function. Its `pause`
function, in turn, invokes the `Root.pause` function to suspend protocol operations. We will shortly delve into the mechanics of this pausing process.
To facilitate the ability to call `Root.pause`, `PauseAdmin` holds the status of a `ward` within the `Root` contract.

### 3.1.2 The `DelayedAdmin`

The `DelayedAdmin` is another concise contract with the capability to pause and unpause the protocol. Additionally, it possesses the ability, after a
`delay` set in the `Root` contract by its deployer (the first ward) and other wards, to designate itself or other Externally Owned Accounts (EOAs) or
contracts as `wards` within the `Root` contract. This delay is enforced due to the `DelayedAdmin` implementing a function to invoke `root.scheduleRely`.
The `root.scheduleRely` function is responsible for scheduling an address to become a `ward` within the `Root` contract.


The `pauseAdmin.scheduleRely` function:

```c++
    // PauseAdmin.sol
    function scheduleRely(address target) public auth {
        root.scheduleRely(target);
    }
```

Which calls the `root.scheduleRely`:

```c++
    // Root.sol
    function scheduleRely(address target) external auth {
        schedule[target] = block.timestamp + delay;
        emit RelyScheduled(target, schedule[target]);
    }
```

As per the documentation provided, the initial setting for this `delay` is 48 hours. Once a rely is scheduled, it can subsequently be
cancelled using the `delayedAdmin.cancelRely` function:

```c++
    // PauseAdmin.sol
    function cancelRely(address target) public auth {
        root.cancelRely(target);
    }
```

That function calls the `root.cancelRely`:

```c++
    // Root.sol
    function cancelRely(address target) external auth {
        schedule[target] = 0;
        emit RelyCancelled(target);
    }
```

Or executed to make it effective by `executeScheduledRely` in the `Root` contract after the `delay` has passed:

```c++
    // Root.sol
    function executeScheduledRely(address target) public {
        require(schedule[target] != 0, "Root/target-not-scheduled");
        require(schedule[target] < block.timestamp, "Root/target-not-ready");

        wards[target] = 1;
        emit Rely(target);

        schedule[target] = 0;
    }
```

### 3.1.3 A Problem with `DelayedAdmin`

Whilst conducting a comparison between the audit contest documentation and the system, I identified a critical discrepancy in the `DelayedAdmin`.
The project team outlined several potential emergency scenarios to aid auditors in comprehending the protocol's security model. One of these
scenarios is outlined below:


>"**Someone controls 1 pause admin and triggers a malicious `pause()`**
>
>* The delayed admin is a `ward` on the pause admin and can trigger `PauseAdmin.removePauser`.
>* It can then trigger `root.unpause()`."

The issue I have identified is that the `DelayedAdmin`, in its current form, does not incorporate any functions to `PauseAdmin.removePauser`.
Given that I believe this deviates from a critical aspect of the protocol's security model, I have chosen to report this discovery with a Medium
severity rating. However, I acknowledge that it could be argued as more appropriate for a Low severity rating if it is ultimately deemed a mere
deviation from the specification. I have expressed my viewpoint in the issue report, and I am confident that it will be evaluated with care
and a comprehensive consideration of the real-world risks it may present, which might diverge from my own assessment.


Following discussions with sponsors to gain deeper insights into the implications of a `DelayedAdmin` unable to execute `pauseAdmin.removePause`,
they exposed two workarounds to address this issue. These workarounds do not alter my stance on the severity within the context of an
audit contest but offer valuable perspectives from the project team to enhance our understanding of the system's security model and its
ability to withstand unforeseen failures.

#### 3.1.3.1 The Delayed Rescue

In a delayed rescue the project team could use the `DelayedAdmin` to:

1. Call `root.scheduleRely` to make a contract or EOA (let's call it `delayedRescuer`) an `ward` in the `Root` contract;
2. Wait the set `delay`, initially be set to 48 hours, to pass;
3. Use `Rescuer` to call `root.relyContract(address(pauseAdmin), address(delayedRescuer))` to become a ward on the `pauseAdmin`;
4. Finally call `pauseAdmin.removePauser(address(maliciousPauser))` to remove the malicious `pauser`.

It is elegant but comes with the drawback of having to wait for the `delay` set in the `Root` contract, which is
not the intended design of this setup.

#### 3.1.3.2 The Prompt Rescue

In the prompt rescue, a contract (let's call it `promptRescuer`) could be made a ward in the `root` contract to:

1. Call `root.relyContract(address(pauseAdmin), address(promptRescuer))`;
2. Call `pauseAdmin.removePauser(address(maliciousPauser))`;
3. Possibly call `root.denyContract(address(promptRescuer))`;

In `Root`, when one contract designates another as its ward, it always introduces certain risks. However, by employing the
Spell pattern, it significantly mitigates at least one of these risks.

## 3.2 The Spell Pattern

Having understood how these two admins fit into the `ward` pattern, and analysing the rescue operation, shows how simple and potentially
powerful the pattern can be. But to realise its power in a more secure manner, we need to enter the Spell pattern.

References to spells can be identified within the codebase's tests, and the sponsors have publicly shared their intentions through the contest's
Discord channel. When utilising `root.relyContract`, it is essential to anticipate that the intended targets for the `wards` in `Root` will
employ the [spell pattern from MakerDAO](https://docs.makerdao.com/smart-contract-modules/governance-module/spell-detailed-documentation).
Besides implementation details and other factors to consider, the utmost detail to retain in the context of this analysis is that
*"a spell can only be cast once"*.

By joining the project's `ward` pattern with the Spell pattern, we can fully understand how critical security operations are expected
to flow throughout the system.

## 3.3 Pausing the Protocol

As mentioned before, the `PauseAdmin` has the responsibility to manage the `pausers` who can pause the protocol. We have seen that the `pauseAdmin`,
triggered by one of its `pausers`, calls the `pause` function in the `Root` contract. The `Root` contract then sets the variable `bool public paused`
to `true`. Here is the `root.pause` function's implementation:

```c++
    // Root.sol
    function pause() external auth {
        paused = true;
        emit Pause();
    }
```

But how can it pause the whole protocol?

The protocol's flow of communication uses a hub-and-spoke model with the `Gateway` in the middle. 
According to the audit's `README` documentation, the `Gateway` contract is an intermediary contract that encodes and decodes
messages using the `Message` contract, and handles routing to and from Centrifuge. This is the graph made available by the project team:

![High Level Contract Overview](https://github.com/code-423n4/2023-09-centrifuge/blob/main/images/contracts.png?raw=true)

The `Gateway` implements the `pauseable` modifier:

```c++
    // Gateway.sol
    modifier pauseable() {
        require(!root.paused(), "Gateway/paused");
        _;
    }
```

This modifier is employed in all critical incoming and outgoing operations within the `Gateway`. Then if `root.paused` is set to `true`, it halts
the "hub" by blocking these operations, thus interrupting the flow of communication in the protocol.

### 3.3.1 Redundancy?

It is important to highlight that both `DelayedAdmin` and `PauseAdmin` have the capability to invoke `root.pause` directly in order to
pause the protocol. What might initially appear as redundancy seems to be, in fact, a well-considered implementation of secure compartmentalization.

`DelayedAdmin` possesses a broader range of powers beyond the functions to `pause` and `unpause` the protocol. In
contrast, `PauseAdmin` primarily oversees `pausers` with the singular purpose of allowing them to pause the protocol. Considering that `DelayedAdmin`
has the potential to cause more harm to the protocol, it should be subject to a stricter oversight, while `pausers` may adhere to a less
stringent regime. While `PauseAdmin` can disrupt the protocol and cause damage by initiating pauses, this impact pales in comparison to what `DelayedAdmin`
could potentially do if not restricted during the `Root`'s `delay`.

### 3.4 Liquidity Pools

Supported by this architecture lies the liquidity pools. These liquidity pools can be deployed on any EVM-compatible blockchain,
whether it's on a Layer 1 or Layer 2, in response to market demand. All of these systems are ultimately interconnected with the Centrifuge chain, which
holds the responsibility of managing the Real-World Assets (RWA) pools:

![Liquidity Pools](https://gist.githubusercontent.com/ciphermarco/438959a403457ab4bb4fc54838cde63c/raw/4a6da758fcb013c6a954d9309634c972034b1a0e/liquidity-pools.png)
(https://gist.githubusercontent.com/ciphermarco/438959a403457ab4bb4fc54838cde63c/raw/4a6da758fcb013c6a954d9309634c972034b1a0e/liquidity-pools.png)

Every tranche, symbolized by a Tranche Token (TT), represents varying levels of risk exposure for investors. For an overview of the authorization relations,
you can refer to the [Ward Pattern section](#31-ward-pattern), and to understand the communication flow, visit the [Pausing the Protocol section](#33-pausing-the-protocol).

### 3.5 Upgrading the Protocol

Another interesting aspect of the architecture is the deliberate choice to use contract migrations instead of upgradeable proxy patterns.
Upgrades to the protocol can be achieved by utilising the Spell pattern to re-organize `wards` and "rely" links across the system. To initiate an upgrade,
a Spell contract is scheduled through the `root.scheduleRely` function. After the specified `delay` period, the upgrade can be executed by using
`root.executeScheduledRely` to complete its tasks in a one-shot fashion. Should the need to cancel the upgrade arise, `root.cancelRely` can be called 
to cancel the scheduled rely operation."

### 3.6 Centralisation Risks

In the context of this analysis, it is imperative to underscore that the usage of "centralisation" specifically pertains to the potential single point of
failure within the project team's assets, such as the scenario where only one key needs to be compromised in order to compromise the whole system.
I am not delving into concerns associated with the likelihood of collusion amongst key personnel.

The most significant concern regarding centralisation lies in the `Root` contract's `wards`. Therefore, it is highly recommended to manage these
`wards` through multi-signature schemes to mitigate the risk of a single point of failure within the system. This encompasses any contract that
could potentially become a `ward` in `Root`, including contracts like `DelayedAdmin` despite its temporary limitation of having to await the `delay`
set in the `Root` contract.

Another noteworthy potential single point of failure lies in the `Router` contract, should it become compromised, as mentioned in the audit's
`README` documentation. This situation can be rectified through a sequence of operations involving the `PauseAdmin` and `DelayedAdmin`, or even
solely relying on the `DelayedAdmin`'s capabilities.

There are other critical entities within the architecture that have the potential to act as single points of failure, leading to varying
degrees of damage. However, for the specific use case at hand, I find the existing tools and safeguards to be adequate for mitigating risks, particularly
when keys are safeguarded through multi-signature schemes and other strategies that require careful consideration. These aspects are beyond the scope of this analysis;
the same applies to what occurs within the Centrifuge chain.

## 4. Implementation Notes

Throughout the audit, certain implementation details stood out as note worthy, and a portion of these could prove valuable for this analysis.

### 4.1 General Impressions

The codebase stood to what is promised in the audit's `README`, so I will just paste it here:

>The coding style of the liquidity-pools code base is heavily inspired by MakerDAO's coding style. Composition over inheritance, no upgradeable
>proxies but rather using contract migrations, and as few dependencies as possible. Authentication uses the ward pattern, in which addresses can be
>relied or denied to get access. Key parameter updates of contracts are executed through file methods.

It was genuinely enjoyable to review this codebase. The code has been meticulously designed to prioritise simplicity. It strikes the right
balance of complexity, avoiding unnecessary complications.

### 4.2 Composition over Inheritance

The codebase's pivotal choice of favoring composition over inheritance, coupled with thoughtful coding practices, has allowed for the creation of
clean and very comprehensible code, free from convoluted inheritance hierarchies that confuse even the project developers themselves. The significance
of this choice in enhancing the security of the protocol cannot be overstated. Clear and well-organised code not only prevents the introduction of
vulnerabilities but also simplifies the process of detecting and resolving any potential issues. And, my opinion is that opting for composition
over inheritance significantly contributed to achieving this objective.

### 4.3 Tests

As mentioned in the [Setup and Tests](LINK LINK LINK), executing the tests using `forge` was effortless, significantly expediting our work as auditors.
Running `forge coverage` also provides a convenient table summarizing the code coverage for these tests:

```
$ forge coverage
[... snip ...]
| File                                  | % Lines           | % Statements      | % Branches       | % Funcs          |
|---------------------------------------|-------------------|-------------------|------------------|------------------|
| script/Axelar.s.sol                   | 0.00% (0/32)      | 0.00% (0/35)      | 0.00% (0/2)      | 0.00% (0/2)      |
| script/Deployer.sol                   | 0.00% (0/40)      | 0.00% (0/42)      | 100.00% (0/0)    | 0.00% (0/4)      |
| script/Permissionless.s.sol           | 0.00% (0/20)      | 0.00% (0/21)      | 0.00% (0/2)      | 0.00% (0/2)      |
| src/Escrow.sol                        | 100.00% (2/2)     | 100.00% (2/2)     | 100.00% (0/0)    | 100.00% (1/1)    |
| src/InvestmentManager.sol             | 97.37% (185/190)  | 96.47% (246/255)  | 61.11% (55/90)   | 97.62% (41/42)   |
| src/LiquidityPool.sol                 | 100.00% (64/64)   | 100.00% (86/86)   | 100.00% (4/4)    | 100.00% (38/38)  |
| src/PoolManager.sol                   | 95.96% (95/99)    | 94.59% (105/111)  | 74.07% (40/54)   | 94.12% (16/17)   |
| src/Root.sol                          | 100.00% (22/22)   | 100.00% (22/22)   | 100.00% (8/8)    | 100.00% (8/8)    |
| src/UserEscrow.sol                    | 100.00% (8/8)     | 100.00% (8/8)     | 100.00% (4/4)    | 100.00% (2/2)    |
| src/admins/DelayedAdmin.sol           | 100.00% (4/4)     | 100.00% (4/4)     | 100.00% (0/0)    | 100.00% (4/4)    |
| src/admins/PauseAdmin.sol             | 100.00% (5/5)     | 100.00% (5/5)     | 100.00% (0/0)    | 100.00% (3/3)    |
| src/gateway/Gateway.sol               | 93.42% (71/76)    | 91.86% (79/86)    | 76.47% (26/34)   | 100.00% (17/17)  |
| src/gateway/Messages.sol              | 59.26% (96/162)   | 59.77% (153/256)  | 16.67% (1/6)     | 55.00% (44/80)   |
| src/gateway/routers/axelar/Router.sol | 100.00% (8/8)     | 100.00% (9/9)     | 100.00% (4/4)    | 100.00% (3/3)    |
| src/gateway/routers/xcm/Router.sol    | 0.00% (0/38)      | 0.00% (0/46)      | 0.00% (0/20)     | 0.00% (0/9)      |
| src/token/ERC20.sol                   | 97.40% (75/77)    | 96.51% (83/86)    | 95.00% (38/40)   | 93.33% (14/15)   |
| src/token/RestrictionManager.sol      | 100.00% (15/15)   | 100.00% (17/17)   | 80.00% (8/10)    | 100.00% (6/6)    |
| src/token/Tranche.sol                 | 100.00% (18/18)   | 100.00% (31/31)   | 100.00% (4/4)    | 75.00% (9/12)    |
| src/util/Auth.sol                     | 100.00% (4/4)     | 100.00% (4/4)     | 100.00% (0/0)    | 100.00% (2/2)    |
| src/util/BytesLib.sol                 | 0.00% (0/28)      | 0.00% (0/28)      | 0.00% (0/16)     | 0.00% (0/7)      |
| src/util/Context.sol                  | 100.00% (1/1)     | 100.00% (1/1)     | 100.00% (0/0)    | 100.00% (1/1)    |
| src/util/Factory.sol                  | 100.00% (24/24)   | 100.00% (37/37)   | 100.00% (0/0)    | 100.00% (3/3)    |
| src/util/MathLib.sol                  | 0.00% (0/30)      | 0.00% (0/38)      | 0.00% (0/6)      | 0.00% (0/3)      |
| src/util/SafeTransferLib.sol          | 100.00% (7/7)     | 100.00% (9/9)     | 66.67% (4/6)     | 100.00% (3/3)    |
| test/TestSetup.t.sol                  | 0.00% (0/49)      | 0.00% (0/72)      | 0.00% (0/6)      | 0.00% (0/9)      |
| test/mock/AxelarGatewayMock.sol       | 100.00% (4/4)     | 100.00% (4/4)     | 100.00% (0/0)    | 100.00% (2/2)    |
| test/mock/GatewayMock.sol             | 2.13% (1/47)      | 2.13% (1/47)      | 100.00% (0/0)    | 9.09% (1/11)     |
| test/mock/Mock.sol                    | 25.00% (1/4)      | 25.00% (1/4)      | 100.00% (0/0)    | 25.00% (1/4)     |
| Total                                 | 65.86% (710/1078) | 66.40% (907/1366) | 62.03% (196/316) | 70.65% (219/310) |
```

Coverage, while not providing a comprehensive view of what is tested and how thoroughly, can serve as a valuable indicator of the
project's commitment to testing practices. Based on the coverage within the audit's scope and my closer assessment of the test
files, I deem the testing practices fitting for this stage of development.

### 4.4 Comments

As previously mentioned, the code is generally clean and strategically straightforward; even so, comments can further enhance the
experience for both auditors and developers. In the context of this codebase, I believe comments are generally effective in
delivering value to readers; however, there are a few sections that could benefit from additional comments.

I expect special attention to more detailed comments in mathematical and pricing operations like, such as those found in 
`InvestmentManager.convertToShares` and `InvestmentManager.converToAssets`. While these operations are not inherently complex,
such comments are crucial because there are two phases of finding bugs in mathematical operations in a code.
Usually, the easier one for most auditors is to spot a discrepancy between the intention the comments documenting the code transmit
and the code itself. The other one is questioning the mathematical foundations and correctness of the formulas used. Being
specific when commenting on the expected outcomes of calculations within the code greatly aids to address the former.

### 4.5 Solidity Versions

When evaluating the quality of a codebase, it is a sensible approach to examine the range of
Solidity versions accepted throughout the codebase:

```
$ grep --include \*.sol --exclude-dir forge-std -hr "pragma solidity" | sort | uniq -c | sort -n
      1 pragma solidity ^0.8.20;
      1 // pragma solidity 0.8.21;
     47 pragma solidity 0.8.21;
```

The majority of our codebase currently relies on version `0.8.21`, a choice that I consider highly advantageous. And, using
`^0.8.20` is obviously not a problem.

Whilst it is true that there are valid points both in favor of and against adopting the latest Solidity version, I believe this
debate holds little relevance for the project at this stage. Opting for the most recent version is unquestionably a superior
choice over potentially risky outdated versions.

## 5. Conclusion

Auditing this codebase and its architectural choices has been a delightful experience. Inherently complex systems greatly benefit from strategically
implemented simplifications, and I believe this project has successfully struck a harmonious balance between the imperative for
simplicity and the challenge of managing complexity. I hope that I have been able to offer a valuable overview of the methodology utilised
during the audit of the contracts within scope, along with pertinent insights for the project team and any party interested in analysing this codebase.






### Time spent:
18 hours
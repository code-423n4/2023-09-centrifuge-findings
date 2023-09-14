## Introduction

Centrifuge is a protocol which works with RWAs and attempts to utilize DeFi's mechanics to it's advantage. The project is a two-way messaging blockchain, with most of it's accountability happening in the deployed chain of choice. Axelar is used as the "layer zero" in here, in order to provide an architecture in which it can execute the exchange said messages.

## Approach taken in evaluating the codebase and evaluation

By starting with a manual review of the code itself, it's clear the codebase is highly dependent on permissioned interactions. With this in mind, attacks which could be coming from the general public were mostly out of question. This also means that in order to find other vulnerabilities, a more systemic approach should be given, as the protocol is dependent on it's bedrock infrastructure. 

## Architecture recommendations

Axelar does not cover many edge case scenarios, and the protocol currently is entirely dependent on it outside EOA action providers. Due to the nature of the protocol, a complete work around may not be possible without refactoring large areas of the code, however in certain cases it should be doable, one example being approaches to deal with an offline sequencer as mentioned in another report.

## Centralization risks

The protocol is highly centralized, mainly due to the nature of the services which are being provided. This is within expectation, as RWA cannot be brought on chain without heavy restrictions in order to comply with security laws. With that said, . An analysis on the centrifuge chain would be required in order to more deeply assert how centralized the chain is, as parachains from polkadot can differ in this regard from one another, however that is outside the scope of this audit.

## Systemic risks

Besides the ones mentioned previously, or in the contest main page, there are a few risks which can be brought from users, rather than developers. Let's say for example an user decides to transfer their token shares to a different address, and claims it to be done by mistake. The sponsors should be able to redeem the user by creating more of these tokens. However a malicious entity could have sent that to an address they already own, doubling their tokens.

There is also a risk of compliancy with the law as well. Recent events have shown that DeFi protocols which attempt to comply with the US law have also a tendency to be persecuted by said laws, such as WOO. As an auditor however, I am not near qualified enough to recommend any sort of prevention or mitigation.

### Time spent:
12 hours
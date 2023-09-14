# Introduction
I decided to take part in auditing Centrifuge because I hadn't encountered a system relating to RWA pools yet. That intrigued me, and although I did not manage to submit any findings, I learnt more about some very interesting topics.

## Analysis of the Codebase

In the first pass I did, everything seemed quite complex and complicated to wrap my head around. After going through all the contracts in scope I understood that the system is not that complex at all but since a lot of the functions relay information through 3-4 contracts until it is finally passed to the router and bridged to the Centrifuge chain, it just seemed very complex.

I created a [diagram](https://pbs.twimg.com/media/F5pxM75W4AAoC3S?format=jpg&name=medium) to explain the architecture to myself a bit better, and it definitely helped me gain a better high-level understanding of numerous things:

1. What "Tranches" and "Tranche Tokens" are
2. How are the Liquidity Pools used in this system and what's their purpose
3. How the profit is generated for the investors
4. The flow of funds in the system

Overall I believe the codebase was tight and well-written. Developers had thought of possible exploits and took preventative measures to counter them.

## Architecture Feedback, Centralization & Systemic Risks
Overall architecture of the system was easy to grasp once you understood how all the calls go from contracts to router. Centralization risks did not seem significant, and there was even a 2nd layer of protection in case a pause admin's account got compromised.

There was one thing that bothered me in the system. With the current implementation, if Alice wants to deposit funds, she first creates a request to deposit, then waits for an epoch to pass (her request gets processed in the CFG Chain at the end of an epoch), then she can can Mint Tranche Tokens, and later down the line if she wants to redeem them back to assets, needs to wait for another epoch execution.

Now that in itself is not an issue, but for context, say that Alice requests a deposit into Tranche A, but reconsiders and wants to take her assets out of the escrow to re-invest them into Tranche B. Well with the current implementation that is impossible. Alice would need to wait for a minimum of 2 epochs to pass. She needs to first wait for deposit epoch, then after she mints and instantly redeems her shares, needs to wait for 2nd redeem epoch to pass, before she can re-invest. Since epoch timeframes are determined by governance on the CFG chain separately for each RWA pool, some epochs could be as long as 1 month.

Either by mistake of entering the wrong Tranche, or reconsidering, Alice might need to wait 2+ months in some scenarios. That just seems inconvenient to me and I feel like there should be added functionality to withdraw your funds from escrow if the epoch has not gone by yet that deposits them.

## Recommendations
I'd recommend adding functionality to cancel a requested deposit and be able to get your assets back from escrow. With the current implementation you'd need to wait for deposit epoch to process them, and then for mint/redeem epoch to give you back your assets. If given Tranche's epoch is 1 month, that's a minimum of 2 months just to re-invest in case you reconsider or made a mistake.

# Time Spent on the Audit
20-30 hours

### Time spent:
25 hours
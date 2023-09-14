# Analysis

**General**

I've spent a few days on this audit and covered the code in scope.

**Approach**

Most of the focus was on deeply understanding the protocol, how the tranching works, i.e junior, senior... additionally time was spent on visualising the protocol from the escrow as a user and also how the ERC 1404 implementation would fit into the developers planned _KYC_ mitigation.

Lastly assumptions were made on how wide adoptions could be for the protocol and what type of issues different wallets, e.g counterfactual wallets could face while integrating with protocol

**Centralization risks**

As usual the admin trusted roles could lead to a lot of issues for any protocol depending on how deceptive an admin gets in the case they ever turn _malicious_

For example, updating prices is restricted to a few, which would lead to issues if not duly followed.

Another issue could be the case of _restricting_ users, i.e - In reality admin can just blacklist anyone _not really someone that's supposed to be restricted_ and nothing would happen instead of having a tangible reason to do this, a user could just be stopped from receiving assets

**Systemic risks**

Using the price from the last epoch seems like not the best practise imo, since currently an epoch could last for an unlimited time frame, meaning underlying assets price might have massively changed within the time frame but pending settlements of withdrawals and deposits would be done with the wrong price, which easily allows users to knowingly/unknowingly position themselves to make the most out of the system

Current ERC 1404 / Restriction implementation is pretty risky and might still lead to regulatory bodies to have a problem with the protocol, this is because current implementation never check if a sender is blacklisted, and as such even if a user is _restricted_ they could still transfer tokens as far as the receiving user is not restricted, this easily stems a lot of issues with regulatory bodies, as even if someone is found to be malicious and say there is a court order to _seize/or curb_ user's assets that can't be done and this could easily land protocol a lawsuit

Overall I'm of the opinion that the Liquidity pooling is an excellent implementation with a few nuances here and there, I do disagree with a few of current implementation and have reported

### Time spent:

~ Around 30-35 hours over the course of duration of the contest... _this includes a few distractions from time to time and around 10 hours of writing reports._


### Time spent:
30 hours
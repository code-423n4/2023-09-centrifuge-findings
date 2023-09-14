1. The increaseAllowance and decreaseAllowance functions in [ERC20.sol](https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/token/ERC20.sol#L139-L159
) needs to be removed.

Reasons;

1. They are not of the standard ERC20
2. Phishing Risks: Their non-standard nature can be exploited by malicious actors to trick users into signing transactions they don't fully understand
3. The decreaseAllowance function is particularly susceptible to front-running attacks, where malicious actors can utilize the allowance before it's decreased.

Read more here: https://github.com/OpenZeppelin/openzeppelin-contracts/issues/4583


2. Hardfork Replay Attacks

It is stated in the doc, that the Protocol is set to deploy to any EVM compatible chain. There is risk of hard fork replay attacks of the chain to be deployed on. Since the best way to mitigate this is to add the chain ID to the signed in permit function of which is catered for already in the [code](https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L216-L237)

Unfortunately, there is still risks of replay attacks most especially the brief moment after post-hardfork where the chains share the same chain ID signatures could be maliciously replayed.


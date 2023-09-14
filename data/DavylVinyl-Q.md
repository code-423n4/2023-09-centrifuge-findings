## Lack of Proper Constructor Access Control
 ### Impact: 
Unauthorized parties can deploy and initialize the contract, potentially leading to malicious deployments or changes to critical contract settings.

### Proof of Concept: 
In the constructor, there is no access control mechanism to restrict who can deploy and initialize the contract. Any address can deploy this contract, which can be risky.

### Recommended Mitigation Steps: 
Implement proper access control in the constructor to ensure that only trusted addresses can deploy and initialize the contract. You can use access control libraries like OpenZeppelin's Ownable or a custom modifier for this purpose.
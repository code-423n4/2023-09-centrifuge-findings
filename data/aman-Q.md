Wrong Comment 
Severity: Informational

The comments in the smart contract indicate that the modifier should validate whether the msg.sender is either the owner or part of the wards. However, upon review, it appears that there is no implemented code within the modifier to manage the wards' flow. This discrepancy needs to be addressed to ensure the contract functions as intended.

Recommendation: We have identified what appears to be an error in the comments section of the smart contract. Consequently, it is necessary to remove the erroneous segment to maintain clarity and accuracy.
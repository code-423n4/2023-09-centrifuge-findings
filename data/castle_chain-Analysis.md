## axelar bridge review 

### adding protection layer to the messaging system 
relaying completely on the validation process of the axelar network has a lot of risk , the validation process can be compromised and lead to a huge damage to the protocol by sending a malicious messages to the core contracts of the system , so it is best practice to add a protection layer to guarantee that the message is not repeated , so it will be better to add a nonce and keep track of it to make sure that there is no repeated message and prevent the malicious validator from gaining control over the important contracts of the system .  

### encode the chain and the source address information with the payload .
the specifying data such as (version , source address , nonce , source chain ) should be encoded with the payload to make sure that the payload is unique of all factors and can not be manipulated or can be predicted , so this will prevent some attacks that depend on passing a malicious payload . 

### a wrong validation process on the router contract will cause the router to revert when receive a messages from the centrifuge chain . 

the implementation of the Router.sol contract assumes that the caller of the function `execute` will be the Axelar gateway contract which is wrong assumption , because according to axelar docs there is a service call the relayer which will be the address who will call the function `execute` on the destination chain , so the protocol should remove this check , and also as per Axelar docs in the standered execute function , there is no check that assume that the caller is the AxelarGateway . 
 

## MATH review 
the protocol uses a math model that accrues the amount of deposit assets and that amount of the tranche tokens that is returned form the centrifuge chain then then the protocol calculate the price by dividing the assets amount over the tranche token amount then **rounding the price down** which result in the depositPrice always be less than what is supposed to be , and this will result in always less amount of currency to be taken from the user and much more amount of the tranche Tokens to be sent to the user which will make the protocol reach a bad state  . 
### the best Math model will be by rounding the depositPrice up , and keep the redeemPrice rounded down 
this implementation should be guarantee that the protocol will not pay more tranche tokens to the user and get less assets for them , and will protect the escrow from insolvency . 
and this is what the uniSwapV3 do in their price calculations . 

## the validation process of the calls that come from the centrifuge chain to the investmentManager should be enhanced . 

the protocol assume that all the call`s parameters and all the values that comes from the centrifuge chain is a valid , but in case of the centrifuge side done down or disabled or even get manipulated by third party , the core contracts of system will get damaged , so I want to recommend adding some validation checks to make sure that the values that comes from the centrifuge chain are correct and reasonable . 
# user experience enhancement  
## allow the users to specify a slippage protection threshold , to protect him against the unexpected prices 

consider adding a slippage protection parameter such as (minAmountOut , or allowancePercentage) to allow the user to specify the maximum change in price that he can afford and to provide a better user experience . 

this threshold can be added in the functions `deposit()` , `mint()` , `redeem()` and `withdraw()` on the liquidity pool contract . 



### Time spent:
60 hours
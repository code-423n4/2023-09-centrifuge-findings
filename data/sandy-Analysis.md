# Centrifuge Protocol - Analysis 

|Head |Details|
|:----------------|:------|
| Approach taken in evaluating the codebase | What is unique? How are the existing patterns used? |
|Codebase quality analysis| Its structure, readability, maintainability, and adherence to best practices|
|Architecture recommendations| The design of the protocol|
|Centralization risks| power, control, or decision-making authority is concentrated in a single entity|
|Other recommendations| Recommendations for improving the quality of your codebase|
|Conclusion| Takeaway from this protocol, recommendations and final review|
|Time spent| Total time spent during auditing and reviewing the codebase |

## Approach taken in evaluating the codebase

Steps:

- ``Use a static code analysis tool``: Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

    - Insecure coding practices
    - Common vulnerabilities
    - Code that is not compliant with security standards

- ``Read the documentation``: The documentation for Centrifuge protocol should provide a detailed overview of the protocol and its codebase. This documentation can be used to understand the purpose of the code and to identify potential areas of concern.

- ``Scope the analysis``: Once you have a basic understanding of the protocol and its codebase, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external APIs, or the code that stores sensitive data.

- ``Manually review the code``: Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

   - Unvalidated user input
   - Hardcoded credentials
   - Insecure cryptographic functions
   - Unsafe deserialization

- ``Mark vulnerable code parts with @audit tags``: Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on.

- ``Dig deep into vulnerable code parts and compare with documentations``:  For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- ``Perform a series of tests``: Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:

  - Valid and invalid user input
  - Different types of attacks
  - Different operating systems and hardware platforms

- ``Report any problems``:  If you find any problems with the code, you should report them to the developers of Centrifuge protocol. The developers will then be able to fix the problems and release a new version of the protocol.

## Codebase quality analysis
The first thing that really stands out in this code base is the lack of inline ``NatSpec`` comments. The code base should use more inline comments for majority of it's functionalities. Apart from that, the code base is well written. Another thing that can be pointed out is lack of zero address checks in the constructor of all the contracts as a simple thing like this can cause to redeploy. The roles and authorization have a main ``root`` which can pause and unpause the whole protocol in case of emergencies which makes it pretty secure. This code base extensively uses interface in each contract and it's implementation is done separately which make the code more readable. This codebase also uses authorization mechanism which uses both contract and EOA for authorization and access control.

## Architecture recommendations

The protocol is designed in an easy to understand way. Most of the users will interact with ``LiquidityPool.sol`` contract which then communicates to ``InvestmentManager.sol`` contract. Also, ``PoolManager.sol`` contract is used to deploy pools and add token. Then, both ``PoolManager.sol`` and ``InvestmentManager.sol`` interact with Centrifuge with the help of messages. These messages are first passed through ``Gateway.sol``, which then are encoded and sent to the ``Router.sol`` contract which routes them to Centrifuge. ``Gateway.sol`` also receives messages from the centrifuge protocol, decodes them and as per message, calls ``Investmentmanager.sol`` and ``PoolManager.sol`` to perform some operations that can only be performed by ``Gateway`` contract. This is the brief overview of how this protocol really works. This is a very unique architectural design and personally, design and architecture wise, I would rate it on A level.

## Centralization risks
There are lot of functionalities in this contract, that can only be accessed by authorized wards. Also, root can pause and unpause the whole protocol but is a single point of failure. But, the protocol is well aware of the centralization risks and has implemented measures to remove compromised wards, routers  and reduce the damage that can be caused compromised or malicious wards and routers.

## Other recommendations

- Regular code reviews and adherence to best practices
- Conduct external audits by security experts
- Consider open sourcing the contract for community review
- Maintain comprehensive security documentation
- Establish a responsible disclosure policy for vulnerabilities
- Implement continuous monitoring for unusual activity
- Educate users about risks and best practices

## Conclusion

Centrifuge is the institutional platform for credit onchain. The protocol is well-structured and uses a decentralized governance model. The codebase is very solid and test coverage is also very good. 

## Time-spent 

40 Hours

### Time spent:
40 hours
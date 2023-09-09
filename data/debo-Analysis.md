Any comments for the judge to contextualize your findings:
I found DOS on this project which I have successfully found before on another project.

Approach taken in evaluating the codebase:
1. Use Mythx to search for bugs.
2. Once medium or high bug found the write exploit code for it.
3. Then submit.

Architecture recommendations:
Utilize AI more.

Codebase quality analysis:
There are some state visibilities not set.
There are also some requirement violations.

Centralization risks:
When lending avoid monopoly by capping individual limits as a percentage, but over time the limit can increase.

Mechanism review:
Most of the code files seem secure enough.  There are recurring security issues with weak random block time stamps and block numbering. And rarely a DOS.  But mostly, the issues are requirement violations and state variables visibility not defined.

Systemic risks:
I do not see a risk of an overall breakdown. Just the odd denial of service on Factory.sol

### Time spent:
15 hours
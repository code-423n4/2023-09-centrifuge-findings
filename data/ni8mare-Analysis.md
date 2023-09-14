The codebase is pretty solid. The tests are well-written and almost cover the whole codebase. The developers have thought of different edge cases. So, it was really difficult to think about vulnerabilities. The number of solidity lines seems to be a lot but it's not really complicated. Most of these lines are used for encoding messages and communication with the Axelar network. But, yes the codebase is really centralised. Not many functions can be called directly by normal users. Most of the functions can only be called by wards or the Axelar gateway. 

One thing that I would like to talk about is the judging of the bot report. The issue l-02, safeApprove() is deprecated, has been wrongly judged. safeApprove has only been deprecated by Openzeppelin. But, the safeApprove used in the code base has been taken from Uniswap and not Openzeppelin. And its implementation is different. So, even if safeApprove is used, it should not be a problem.

### Time spent:
12 hours
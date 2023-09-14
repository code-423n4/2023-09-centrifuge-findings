* `handle` function of Gateway.sol plays a very important role on this contract.
One thing that is bothering me is that,there is no events declared inside those else if conditions.
As it does important state-changes,an event should be fired on which condition it fulfills.
So,adding events accordingly inside the if/else if conditions is recommended to let the users as well as off chain applications know about any particular update.

* There are some functions which are named exactly the same and takes similar input parameters but their return value is different(such as `formatUpdateMember`,`formatDomain` of Messages.sol).
Suggesting to set the name of the functions slightly differently to remove any possible confusions regarding the functions.

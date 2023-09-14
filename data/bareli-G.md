https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/Escrow.sol

L 7-9:interface ApproveLike {
    function approve(address, uint256) external returns (bool);
} Why is this required?

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/UserEscrow.sol
L30:  destinations[token][destination] += amount; it should not be += as it takes more gas.

https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Auth.sol
L8:  mapping(address => uint256) public wards;  it should be mapping(address=>bool) .
https://github.com/code-423n4/2023-09-centrifuge/blob/main/src/util/Factory.sol
L47:for (uint256 i = 0; i < wards.length; i++) { we should assign wards.length to cached variable.
L103:  for (uint256 i = 0; i < trancheTokenWards.length; i++) { same as above.
L117: for (uint256 i = 0; i < restrictionManagerWards.length; i++) { Same as above.
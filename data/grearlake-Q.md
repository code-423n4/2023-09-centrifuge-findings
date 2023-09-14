1, There might not enough currency token for Centrifuge chain for success call handleExecutedCollectRedeem function
When Centrifuge chain send message to pool, Gateway#handle will handle message:

          } else if (Messages.isExecutedCollectRedeem(message)) {
            (
                uint64 poolId,
                bytes16 trancheId,
                address investor,
                uint128 currency,
                uint128 currencyPayout,
                uint128 trancheTokensPayout
            ) = Messages.parseExecutedCollectRedeem(message);
            investmentManager.handleExecutedCollectRedeem(
                poolId,
                trancheId,
                investor,
                currency,
                currencyPayout,
                trancheTokensPayout
            );

In InvestmentManager#handleExecutedCollectRedeem, it will try to transfer currency from escrow to recipient that request in previous escrow:

        userEscrow.transferIn(
            _currency,
            address(escrow),
            recipient,
            currencyPayout
        );

But problem is there is no guardian that escrow will have enough, since ward in contract is able to taken out by ward, and ward can be set to any address by root because escrow is rely on root, as it is described in Deployer.sol:

            Escrow(address(escrow)).rely(address(root));
Moreover, there even is a chance that all currency that inside escrow can be stolen because of malicious ward in escrow that set by root.

To mitigration, escrow should be independent, not able to be fully controlled by any address for safe, and make sure that escrow have enough tokens to transfer for user when epoch end

2, Should not use draft proposal
RestrictionManager.sol use ERC1404, which is draft proposal (https://github.com/ethereum/eips/issues/1404), which is not recognized. Instead, can use others like (https://eips.ethereum.org/EIPS/eip-1450)

3, Ward in InvestmentManager can mint unlimited tranche token
ERC20#mint() is used to mint token to address:

    function mint(address to, uint256 value) public virtual auth {
        require(
            to != address(0) && to != address(this),
            "ERC20/invalid-address"
        );
        unchecked {
            balanceOf[to] = balanceOf[to] + value; // note: we don't need an overflow check here b/c balanceOf[to] <= totalSupply and there is an overflow check below
        }
        totalSupply = totalSupply + value;

        emit Transfer(address(0), to, value);
    }
If any person that have ward that set by root in Root#relyContract(), they can call this function to mint unlimited tranche token to any address. To mitigrate this, make sure address called to this function is only contract address that will call to this function

5, Tranche token name only can up to 128 character, tranche token symbol only can up to 32 character
When update tranche token metadata, Messages#formatUpdateTrancheTokenMetadata is called:

    function formatUpdateTrancheTokenMetadata(
        uint64 poolId,
        bytes16 trancheId,
        string memory tokenName,
        string memory tokenSymbol
    ) internal pure returns (bytes memory) {
        // TODO(nuno): Now, we encode `tokenName` as a 128-bytearray by first encoding `tokenName`
        // to bytes32 and then we encode three empty bytes32's, which sum up to a total of 128 bytes.
        // Add support to actually encode `tokenName` fully as a 128 bytes string.
        return
            abi.encodePacked(
                uint8(Call.UpdateTrancheTokenMetadata),
                poolId,
                trancheId,
                _stringToBytes128(tokenName),
                _stringToBytes32(tokenSymbol)
            );
    }

`_stringToBytes128(tokenName)` mean only 128 characters is saved, if more than 128 character, the rest will be ignored, same with `_stringToBytes32(tokenSymbol)`

6, Strict checking condition in UserEscrow#transferOut
In UserEscrow#transferOut(), there is a require check that explained by user:

            /// @dev transferOut can only be initiated by the destination address or an authorized admin.
            ///      The check is just an additional protection to secure destination funds in case of compromized auth.
            ///      Since userEscrow is not able to decrease the allowance for the receiver,
            ///      a transfer is only possible in case receiver has received the full allowance from destination address.
But this checking condition could also make user hard to directly transfer redeem to other user. To directly transfer, they have approve from outside of contract, which increase risk of losing token by approve() full balance to other address
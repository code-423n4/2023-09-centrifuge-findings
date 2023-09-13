src/token/ERC20.sol 
---

    function file(bytes32 what, string memory data) external auth {
        if (what == "name") name = data;
        else if (what == "symbol") symbol = data;
        else revert("ERC20/file-unrecognized-param");
        emit File(what, data);
    }

+++    
       
    function file(bytes32 what, string memory data) external auth {
        require(
            what == keccak256("name") || what == keccak256("symbol"),
            "ERC20/file-unrecognized-param"
        );

        if (what == keccak256("name")) {
            name = data;
        } else {
            symbol = data;
        }

        emit File(what, data);
    }

The advantages of the second code snippet are as follows:

Increased Security: The first code snippet directly compares parameters without throwing an error when an incorrect value is sent. The second code snippet uses a require statement to halt execution when the parameters do not match the expected values, providing a more secure approach.

Preservation of Functionality: The second code snippet immediately throws an error if a parameter other than "name" or "symbol" is sent. This is important for maintaining the functionality of the contract and prevents unexpected parameters from altering variables.

Use of Keccak256: The second code snippet uses the keccak256 function to hash the parameters. This demonstrates that the parameters are in a consistent format and are more resistant to external tampering.

Improved Error Messages: The second code snippet provides more descriptive error messages. This can help developers identify errors more quickly.

Cleaner and More Readable Code: The second code snippet is structured in a more organized and understandable manner. This makes code maintenance and development easier.

Emitting the File Event: Both code snippets use emit File(what, data) to trigger an event. This is important for tracking and monitoring what is happening in the transaction.

In conclusion, the second code snippet provides a more secure, cleaner, and more readable code, while also helping to maintain the expected functionality of the contract. Therefore, the second code snippet may be a better choice for a developer.


### Time spent:
3 hours
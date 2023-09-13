## Impact
Using temporary variables to update balances is a dangerous construction that has led to several hacks in the past.
---
https://github.com/code-423n4/2023-09-centrifuge/blob/512e7a71ebd9ae76384f837204216f26380c9f91/src/token/ERC20.sol#L90C1-L130C1
 
// --- ERC20 Mutations ---
---
    function transfer(address to, uint256 value) public virtual returns (bool) {
        require(to != address(0) && to != address(this), "ERC20/invalid-address");
        uint256 balance = balanceOf[_msgSender()];
        require(balance >= value, "ERC20/insufficient-balance");

        unchecked {
            balanceOf[_msgSender()] = balance - value;
            balanceOf[to] += value;
        }

        emit Transfer(_msgSender(), to, value);

        return true;
    }

    function transferFrom(address from, address to, uint256 value) public virtual returns (bool) {
        require(to != address(0) && to != address(this), "ERC20/invalid-address");
        uint256 balance = balanceOf[from];
        require(balance >= value, "ERC20/insufficient-balance");

        if (from != _msgSender()) {
            uint256 allowed = allowance[from][_msgSender()];
            if (allowed != type(uint256).max) {
                require(allowed >= value, "ERC20/insufficient-allowance");
                unchecked {
                    allowance[from][_msgSender()] = allowed - value;
                }
            }
        }

        unchecked {
            balanceOf[from] = balance - value;
            balanceOf[to] += value;
        }

        emit Transfer(from, to, value);

        return true;
    }

## Recommended Mitigation Steps

// --- ERC20 Mutations ---
---
    function transfer(address to, uint256 value) public virtual returns (bool) {
        ....
        
        unchecked {
            balanceOf[_msgSender()] -= value;
            balanceOf[to] += value;
         }
    
        ....
     }


    function transferFrom(address from, address to, uint256 value) public virtual returns (bool) {
        ....

        unchecked {        
            balanceOf[from] -= value;
            balanceOf[to] += value;
        }
    
        ....
     }
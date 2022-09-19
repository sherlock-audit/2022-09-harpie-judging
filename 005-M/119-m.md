defsec
# Deflationary tokens are not handled uniformly across the protocol

## Summary

The code base do not support `rebasing/deflationary/inflationary`  tokens whose balance changes during transfers or over time.

## Vulnerability Detail

The code base do not support `rebasing/deflationary/inflationary`  tokens whose balance changes during transfers or over time. The necessary checks include at least verifying the amount of tokens transferred to contracts before and after the actual transfer to infer any fees/interest. 


## Impact

Vault receives lower amount than excepted.

## Code Snippet

```
    function transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool) {
        require (_transferEOAs[msg.sender] == true || msg.sender == address(this), "Caller must be an approved caller.");
        require(_erc20Address != address(this));
        // Do the functions after the following line occur if the following line fails? Does it revert? Test
        uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
        IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
        (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
        );
        require(loggingSuccess, string (loggingResult));
        emit successfulERC20Transfer(_ownerAddress, _erc20Address);
        return loggingSuccess;
    }
```

## Tool used

Manual Review

## Recommendation

Consider checking previous balance/after balance equals to amount for any `rebasing/inflation/deflation` reward tokens.


saian
# Contract doesn't handle fee-on-transfer or rebasing tokens

## Vulnerability Detail

The `transferERC20` transfers user's token amount to the vault, If the token is fee-on-transfer, the amount of tokens transferred to the vault will be lesser than the value updated in the vault.
This will cause users to loose assets if other users withdraw the same token from the vault, or revert due to the unavailibility of tokens.
If the token is a rebasing token, rewards accrued to the vault address cannot be withdrawn by the users

https://github.com/sherlock-audit/2022-09-harpie-saianmk/blob/924f1e416faa108d28b798d52b3308bcb9f27093/contracts/contracts/Transfer.sol#L88

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

https://github.com/sherlock-audit/2022-09-harpie-saianmk/blob/924f1e416faa108d28b798d52b3308bcb9f27093/contracts/contracts/Vault.sol#L82

```
    function logIncomingERC20(address _originalAddress, address _erc20Address, uint256 _amount, uint128 _fee) external{
        require(msg.sender == transferer, "Only the transferer contract can log funds.");
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee += _fee;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
    }
```

## Tool used

Manual Review

## Recommendation

Consider using a variable to track the previous balance of ERC20 tokens in the vault, add difference of amounts to the user's mapping 

Add function to withdraw excess tokens from the vault
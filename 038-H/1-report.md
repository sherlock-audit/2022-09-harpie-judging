cryptphi
# Recipient could lose funds due to unchecked return value of ERC20 transfer

## Summary
 [Vault.withdrawERC20()](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L118) ignores the return value of IERC20.transfer() , which may cause a loss of funds for recipients since the `amountStored` value has been updated to 0 during the withdrawERC20() call.

## Vulnerability Detail
When the recipient calls [Vault.withdrawERC20()](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L118) , the function attempts to transfer the `amountStored` to the receiver. However, in the event of a failed IERC20.transfer() call, there is no check on the return value. This would cause a loss for the recipient as the recipient no longer has any amountStored.

## Impact
Loss of recipient funds to contract

## Code Snippet
```
function withdrawERC20(address _originalAddress, address _erc20Address) payable external {
        // Function caller is the recipient
        // Check that function caller is the recipientAddress
        require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
        require(canWithdrawERC20(_originalAddress, _erc20Address) > 0, "No withdrawal allowance.");
        require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
        // TODO: Change this to withdrawing the entire amount
        uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored = 0;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = 0;
        IERC20(_erc20Address).transfer(msg.sender, amount);
    }
```

## Tool used

Manual Review

## Recommendation
Add a require check for successful transfer of ERC20 tokens.

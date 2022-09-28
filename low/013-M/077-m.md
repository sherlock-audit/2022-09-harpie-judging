rbserver
# Recipient can lose extra ETH that is sent when calling Vault.withdrawERC20 function

## Summary
When calling the `Vault.withdrawERC20` function, the recipient can lose the extra ETH, which is more than the fee, that is sent.

## Vulnerability Detail
In the `Vault.withdrawERC20` function that is shown in the Code Snippet section, `require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.")` is executed. It is possible that the `msg.sender`, who is the recipient, sends more ETH than `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee` when calling `Vault.withdrawERC20`. The extra ETH that is sent will not be refunded to the recipient.

## Impact
Because that the extra ETH that is sent will not be refunded, the recipient loses these ETH amount. Disputes can occur when the recipient asks the protocol for a refund afterwards.

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L118-L128
```solidity
    function withdrawERC20(address _originalAddress, address _erc20Address) payable external {
        require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
        require(_erc20Address != address(this), "The vault is not a token address");
        require(canWithdrawERC20(_originalAddress, _erc20Address) > 0, "No withdrawal allowance.");
        require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");

        uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored = 0;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = 0;
        IERC20(_erc20Address).safeTransfer(msg.sender, amount);
    }
```

## Tool used

Manual Review

## Recommendation
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L122 can be updated to the following code.
```solidity
        require(msg.value == _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
```
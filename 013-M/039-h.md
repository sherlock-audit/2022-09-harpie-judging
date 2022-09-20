cryptphi
# Contract does not return excess fee paid

## Summary
When a receiver calls [Vault.withdrawERC20() ](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L118) or [Vault.withdrawERC721()](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L129), they could send ETH which is more than the fee amount. However, the contract's functions do not make provision for returning excessive ETH sent.

## Vulnerability Detail
The [Vault.withdrawERC20()](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L118) and [Vault.withdrawERC721()](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L129) are payable functions which  checks that the `msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee` , but does not make provision to send back to the caller any excessive eth sent.

## Impact
User can lose funds to contract.

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

    function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {
        require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
        require(canWithdrawERC721(_originalAddress, _erc721Address, _id), "Insufficient withdrawal allowance.");
        require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
        _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].isStored = false;
        _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee = 0;
        IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
    }
```

## Tool used
Manual Review

## Recommendation
Add an additional logic to transfer any excessive ETH back to the caller
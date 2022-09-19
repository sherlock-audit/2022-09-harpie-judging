rbserver
# Recipient can lose extra ETH that is sent when calling Vault.withdrawERC721 function

## Summary
When calling the `Vault.withdrawERC721` function, the recipient can send and lose the extra ETH that is more than the fee.

## Vulnerability Detail
Calling the `Vault.withdrawERC721` function that is shown in the Code Snippet section executes `require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.")`. When calling `Vault.withdrawERC721`, it is possible that the `msg.sender`, who is the recipient, sends more ETH than `_erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee` . The extra ETH that is sent will not be refunded to the recipient.

## Impact
The recipient loses the extra ETH that is sent since it will not be refunded. Later, disputes can occur when the recipient asks the protocol for a refund.

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L129-L138
```solidity
    function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {
        require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
        require(_erc721Address != address(this), "The vault is not a token address");
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
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L133 can be updated to the following code.
```solidity
        require(msg.value == _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```
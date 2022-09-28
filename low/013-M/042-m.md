Chom
# Not returning leftover ETH if users send more ETH than their fee.

## Summary
Not returning leftover ETH if users send more ETH than their fee.

## Vulnerability Detail

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

If msg.value > xxx.fee for example xxx.fee is 100 but sending msg.value as 110, it will keep 110 as fee instead of 100. Leftover 10 ETH isn't returned to the user.

## Impact
Users may lose their ETH if they overpaid the fee.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-Chomtana/blob/77dc382dae7542c37dfeecaeb4739f2240c4cdd1/contracts/contracts/Vault.sol#L118-L138

## Tool used

Manual Review

## Recommendation
Add ETH refunding mechanism or require users to send msg.value equal to the fee (Using == instead of >=)

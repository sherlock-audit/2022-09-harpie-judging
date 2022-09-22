csanuragjain
# Excess ETH not refunded

## Summary
It was observed that while calling withdrawERC20/withdrawERC721, there was no refund given if user has passed ETH>fee. This means user will lose funds

## Vulnerability Detail
1.  Assume _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee is currently set to 10
2. Admin decides to reduce the fee using reduceERC20Fee for this user making it _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = 5
3. _originalAddress, who is unaware of this change simply calls withdrawERC20 function 
4. Withdrawal is complete and _originalAddress is charged amount 10 instead of 5. Ideally 10-5 amount should have been refunded back to user

## Impact
User will lose funds

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L122
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L133

## Tool used
Manual Review

## Recommendation
Refund the excess ETH which is transferred to contract by user
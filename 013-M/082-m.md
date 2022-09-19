TomJ
# User may accidentally overpay fee in withdrawERC20() and withdrawERC721()

## Summary
withdrawERC20() and withdrawERC721() allows user to pay ETH equal to or greater than the fee.
It is possible for an user to overpay the fee than he/she actually had to pay.

## Vulnerability Detail
`withdrawERC20()`  and `withdrawERC721()` allows `msg.value > fee`

## Impact
User overpaying the fee than he/she actually had to pay. (loss of funds)

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L122
```solidity
Vault.sol
122:        require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
```

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L133
```solidity
Vault.sol
133:        require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```

## Tool used
Manual Review

## Recommendation
Consider changing the check so that it is checking msg.value is exactly equal to the fee.
```solidity
require(msg.value == _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
require(msg.value == _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```

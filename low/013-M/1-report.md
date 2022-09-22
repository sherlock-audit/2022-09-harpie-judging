0xNazgul
# [NAZ-M2] Overpayment of Native ETH Is Not Refunded To Buyer

## Summary
A user of the `Vault.sol` contract can accidentally send more ETH than needed. 

## Vulnerability Detail
The user who wants to retrieve their ERC20/ERC721 tokens back from the vault have to pay a fee in native ETH. The fee is 7% fee on any asset that was successfully recovered. When the user overpays the fee it is accepted and the excess is not returned.

## Impact
The overpaying of ETH from a user can make it hard to refund and can cause unwanted issues.

## Code Snippet
[`Vault.sol#L122`](https://github.com/sherlock-audit/2022-09-harpie-0xNazgul/blob/master/contracts/contracts/Vault.sol#L122), [`Vault.sol#L133`](https://github.com/sherlock-audit/2022-09-harpie-0xNazgul/blob/master/contracts/contracts/Vault.sol#L133)

## Tool used
Manual Review

## Recommendation
Consider modifying the check such that the `msg.value` is exactly equal to the fee e.g.
```js
require(msg.value == _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
&&
require(msg.value == _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```
Or to be more fluid subtract the `msg.value` sent from the fee and transfer it back.
hickuphh3
# Excess withdrawal fees are taken by protocol

## Summary

Should a user send more ETH than necessary for the withdrawal fee, they are not refunded; the protocol unfairly takes the full amount.

## Vulnerability Detail

Users that send more than the required withdrawal fee do not get refunded. Instead, the protocol will take the full `msg.value` sent for the withdrawal transaction.

## Impact

Users overpay the withdrawal fee, causing them to feel rugged.

## Code Snippet

[https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/7ef0e49d57918264f8049af46ba8738b77f2cbe2/contracts/contracts/Vault.sol#L122](https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/7ef0e49d57918264f8049af46ba8738b77f2cbe2/contracts/contracts/Vault.sol#L122)

[https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/7ef0e49d57918264f8049af46ba8738b77f2cbe2/contracts/contracts/Vault.sol#L133](https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/7ef0e49d57918264f8049af46ba8738b77f2cbe2/contracts/contracts/Vault.sol#L133)

## Recommendation

1) Ensure strict equality

```diff
- require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
+ require(msg.value == _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");

- require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
+ require(msg.value == _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```

This however runs the risk of griefing for ERC20 withdrawals (unlikely, but possible): the protocol frontruns the withdrawal transaction with a recovery transaction to increase the withdrawal fee to be charged, causing the user’s withdrawal transaction to revert and waste gas.

2) Refund excess

Transfer excess ETH back to the user.
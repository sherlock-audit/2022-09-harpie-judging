hickuphh3
# Funds can be held hostage due to uncapped fee

## Summary

The lack of an upper bound on the recovery fee allows the protocol to hold users’ assets for a hefty ransom.

## Vulnerability Detail

A 7% recovery fee is charged on any asset that was successfully recovered. This payment is taken in ETH and is taken upon withdrawing the recovered assets out of the vault. 

Given how nascent the protocol is, it would be advisable to set an absolute cap on the maximum fee that can be charged (either accumulatively or per recovery transaction). The cap can be increased or removed after some time, when reputation has been established.

This gives users some assurance that the protocol wouldn’t arbitrarily “recover” their assets without due cause, and slap a hefty recovery fee for profit.

## Tool Used

Manual Review

## Recommendation

Have a `feeLimit` that is checked in the `logIncomingERC20()` and `logIncomingERC721()` functions.

```solidity
// Option 1: check per recovery transaction
if (_fee > feeLimit) _fee = feeLimit;

// Option 2: limit accumulated fees
uint256 accruedFee = _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee + _fee;
if (accruedFee > feeLimit) accruedFee = feeLimit;
_erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = accruedFee;
```
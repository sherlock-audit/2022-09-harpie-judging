sach1r0
# Use `call()` instead of `transfer()` when transferring eth

## Summary
The `withdrawPayments()` function is using the old `transfer()` method instead of the safer `call()`.

## Vulnerability Detail
`transfer()` method only allows the recipient to use 2300 gas. Transfer will fail if ever the recipient uses more than that gas. Use `call()` instead of `transfer()` when transferring eth. However, keep in mind to follow the checks-effects-interactions pattern to reduce the risk of reentrancy when using `call()` function.
See reference for similar issue: https://github.com/code-423n4/2022-05-rubicon-findings/issues/82

## Impact
Transfer will fail if ever the recipient uses more than 2300 gas.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-sach1r0/blob/163a3c399e6c26270cbc90250f1ef202444f3f0e/contracts/contracts/Vault.sol#L159

## Tool used
vim
Manual Review

## Recommendation
I recommend using `call()` method instead of `transfer()` when transferring ETH.
```
(bool success, ) = feeController.call{value: amount}("");
require(success, "Transfer Failed");
```

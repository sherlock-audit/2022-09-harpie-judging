IllIllI
# No support for fee-on-transfer tokens

## Summary
The vault accounting does not include support for fee-on-transfer tokens

## Vulnerability Detail
Fee-on-transfer tokens deduct a small amount of fees during every transfer, meaning the amount passed to the `transfer()` or `transferFrom()` is not the amount that ends up in the recipient's balance. The vault doesn't account for this, because it relies on the amount stated by the log messages, which is the original amount

## Impact
Users that get their funds sent to the vault are told that they are allowed to withdraw more funds than were actually saved. At the very end, or if there is a large whale, the total amount of fees that have been deducted will mean that users, even with small amounts saved, will not be able to withdraw their funds, since the actual vault balance will not match the vault accounting. This will affect many users and is therefore high risk.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L96-L98

The above function uses the original customer balance stored during the transfer, not the actual balance transferred
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L92-L100


## Tool used

Manual Review

## Recommendation
Measure the amount before and after the transfer, and store the difference as the amount, rather than the amount stated
IllIllI
# Multiple attacks that add up to more than `type(uint128).max` are not protected against

## Summary
The amount accounting may overflow, causing transactions to revert

## Vulnerability Detail
The total amount transferred is stored in a `uint128` which may overflow over multiple transactions

## Impact
Once the variable reaches `type(uint128).max` additions to the amount will cause the value to overflow, causing the Harpie front-running transaction to fail, leaving the user unprotected.

If a token has a lot of decimals, or is highly inflationary, it's possible for this to happen. A similar issue exists with the fee and could happen if Ether hyper-inflates.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L82-L86
## Tool used

Manual Review

## Recommendation
Use `uint256` for tracking the amounts transferred, and for the fees
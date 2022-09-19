IllIllI
# Batched transfers do not take into account the block gas size

## Summary
Batched transfers do not take into account the block gas size

## Vulnerability Detail
If there are too many transactions in the batch, or one of the transactions takes up a surprisingly large amount of gas, the block will run out of gas, causing the transaction to revert

## Impact
The transaction will revert, and the attacker's transaction will take effect, with the user losing funds

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L108-L117

## Tool used

Manual Review

## Recommendation
Do not batch transfers, since it's not possible to know ahead of time how much gas will be used
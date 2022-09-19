IllIllI
# Token amounts over `type(uint128).max` are lost forever

## Summary
Token transfers where the amount transferred is larger than `type(uint128).max` have their higher-order bits truncated. If a token is highly inflationary, or has a large number of decimals, it's easy for a user to hit this limit.

## Vulnerability Detail
Amounts passed in as `uint256` are cast to `uint128` before being stored.

## Impact
When it's time to withdraw, the user is only able to withdraw the amount that was stored corresponding to the lower-order bits, which if the amount was truncated, means they lose at least the majority of the tokens.

## Code Snippet

https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L81-L86

See line 85 above

## Tool used
Manual Review

## Recommendation
Use `uint256` for `amountStored`
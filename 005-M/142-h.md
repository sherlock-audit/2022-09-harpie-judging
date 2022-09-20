IllIllI
# No support for rebasing tokens

## Summary
Rebasing tokens are tokens that change the answer to `balanceOf()` questions, over time. One such rebasing token is an Aave token

## Vulnerability Detail
The vault stores the amount initially transferred in, but does not account for rebasing interest payments, or airdrops that occur while the funds are in the vault

## Impact
Users will lose any rebasing funds or airdrops that are owed to them while the funds are in the vault, and there's no way to take the excess out, so the funds will be locked forever. This affects a lot of users since these tokens (e.g. Aave tokens) are common, so it is therefore high risk.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L96-L98

only the amount initially stored is available for withdrawal

## Tool used

Manual Review

## Recommendation
Keep track of shares of the total balance, rather than specific token amounts
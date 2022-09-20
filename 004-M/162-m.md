IllIllI
# Cross-chain replay attacks are possible with `changeRecipientAddress()`

## Summary
Mistakes made on one chain can be re-applied to a new chain

## Vulnerability Detail
There is no `chain.id` in the signed data

## Impact
If a user does a `changeRecipientAddress()` using the wrong network, an attacker can replay the action on the correct chain, and steal the funds a-la the wintermute gnosis safe attack, where the attacker can create the same address that the user tried to, and steal the funds from there

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L60-L73

## Tool used

Manual Review

## Recommendation
Include the `chain.id` in what's hashed

IllIllI
# The vault steals any excess Ether sent to it

## Summary
The vault steals any excess Ether sent to it

## Vulnerability Detail
The withdrawal functions allow the user to send more Ether than is required to cover the fee, without returning the excess

## Impact
Users will pay more than they need to, and get nothing in return

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L133
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L122

## Tool used

Manual Review

## Recommendation
Require that `msg.value` exactly equals the fee
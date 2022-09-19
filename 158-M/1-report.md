IllIllI
# Transfer hooks may prevent Harpie front-running transactions from working

## Summary
There are tokens like ERC777 tokens that have transfer hooks, which are also ERC20-compliant. ERC721 tokens also have transfer hooks, which can ensure that royalties are paid.

## Vulnerability Detail
If a token has a transfer hook that prevents a transaction from completing if an extra step is not completed along with the `transfer()` call (e.g. the payment of royalties), then the Harpie front-running transaction may revert, while the attacker's goes through, because they paid the royalty

## Impact
Users will be unprotected if transfer hooks add other conditions that Harpie cannot satisfy

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L57-L70
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L88-L104

## Tool used

Manual Review

## Recommendation
Explicitly state accounts that implement non-empty transfer hooks are not supported
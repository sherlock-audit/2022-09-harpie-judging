IllIllI
# Compromised transferrer EOAs can transfer tokens without any constraints

## Summary
EOAs use single private keys which can be and often are compromised.

## Vulnerability Detail
The Transfer contract allows the giving of _many_ different EOAs, the ability to arbitrarily transfer user funds. All it takes is for one key to be compromised for all user funds to be moved to the vault. 

## Impact
If the moved tokens are a fee-on-transfer token, the users affected would be docked those fees, even if the funds are transferred back later. There will also be massive reputational damage done to Harpie itself.

## Code Snippet
Any EOA, or even any contract address can be approved:
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L119-L123

And once approved, can transfer funds from any approved account:
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L88-L89

## Tool used

Manual Review

## Recommendation
Users should be partitioned into subsets, where each approved EOA is only allowed to transfer for that specific subset (e.g. on a per-token and per-address range basis), and a second factor should be used, such as also requiring a second signature by a global Harpie private key 
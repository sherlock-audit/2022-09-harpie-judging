IllIllI
# An NFT may be thought to be an ERC20, and be lost

## Summary
Not all ERC721 tokens properly follow the standard.

## Vulnerability Detail
If the automation that identifies whether the token is an ERC20 or a ERC721 gets confused, the wrong action may be taken, leading to transactions reverting, or the NFT being stuck in the vault

## Impact
A Harpie front-running transaction may fail, (e.g. because the NFT was mistaken for an ERC20, and was included in a batch, and caused that part of the batch run out of gas and fail), leading to the protection not working

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L111

## Tool used

Manual Review

## Recommendation
Be explicit in the documentation about what constitutes an ERC721, and provide ways for users to check that the NFT is considered an ERC721, and not an ERC20
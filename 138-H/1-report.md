IllIllI
# No support for newer NFTs

## Summary
Many of the newer NFTs user the ERC115 standard, rather than the ERC721 standard

## Vulnerability Detail
Harpie only has transfer hooks for ERC20 and ERC721 tokens

## Impact
A user that thinks their NFTs are protected is not in fact protected, if the NFT uses ERC1155

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L57-L117

There is no non-safe version of the ERC1155 transfer function, so the Vault will be unable to receive any such NFTs, and the front-running will not be able to be done

## Tool used

Manual Review

## Recommendation
Add explicit support for ERC1155

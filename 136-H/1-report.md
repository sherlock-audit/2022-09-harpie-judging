IllIllI
# No punk/rock support

## Summary
No support for crypto punk/rock NFTs

## Vulnerability Detail
The project documentation talks about keeping a user's tokens/NFTs safe, but doesn't protect NFTs that came before the ERC721 standard

## Impact
A user that thinks they're protected and has Punks and Rocks can have those NFTs stolen. These are some of the most valuable NFTs, so it's high risk for a user to think they're protected when they're not

## Code Snippet
Nowhere in the documentation does it state that only ERC721 tokens are supported, and the only transfer functions are for ERC20 and ERC721s

https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L57-L117

## Tool used

Manual Review

## Recommendation
Instruct users to wrap their non-standard NFTs with ERC721 wrappers
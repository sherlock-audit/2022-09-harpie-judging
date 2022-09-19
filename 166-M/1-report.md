IllIllI
# NFTs may be lost due to non-safe transfers

## Summary
NFTs may be lost due to non-safe transfers

## Vulnerability Detail
The code uses `transferFrom()` rather than `safeTransferFrom()` so no checks are done about whether the recipient can handle NFTs

## Impact
The user may use an address for the `changeRecipientAddress()` call, which may be unable to handle NFTs, which will cause the NFTs to be locked once they are transferred

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L137-L137

## Tool used

Manual Review

## Recommendation
Use `safeTransferFrom()` for NFTs
Chom
# Using ERC721 unsafe transferFrom will cause some NFT loss if the recipient can't accept ERC721 and won't execute onERC721Received in the recipient contract.

## Summary
Using ERC721 unsafe transferFrom will cause some NFT loss if the recipient can't accept ERC721 and won't execute onERC721Received in the recipient contract.

## Vulnerability Detail
function transferERC721 in Transfer and withdrawERC721 in Valult contract is using unsafe transferFrom.

## Impact
Cause some NFT loss if the recipient can't accept ERC721 and won't execute onERC721Received in the recipient contract. The user can't write logic to handle received erc721 if onERC721Received is not called.

## Code Snippet

https://github.com/sherlock-audit/2022-09-harpie-Chomtana/blob/77dc382dae7542c37dfeecaeb4739f2240c4cdd1/contracts/contracts/Transfer.sol#L60-L63

https://github.com/sherlock-audit/2022-09-harpie-Chomtana/blob/77dc382dae7542c37dfeecaeb4739f2240c4cdd1/contracts/contracts/Vault.sol#L137

## Tool used

Manual Review

## Recommendation
Use safeTransferFrom instead of transferFrom.
IEatBabyCarrots
# Use safeTransferFrom for sending ERC721 tokens

## Summary
The `withdrawERC721` function uses `transferFrom` instead of `safeTransferFrom` to send ERC721 tokens. 

## Vulnerability Detail
If `msg.sender` is a contract that does not support ERC721 tokens, this results in the NFT locked up in an address from which you can never retrieve it

## Impact
A user may accidentally make their NFT inaccessible

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-Fluffy9/blob/b391dc56dadb676fef50af6a87b4e12027326d8e/contracts/contracts/Vault.sol#L129-L138

## Tool used

Manual Review

## Recommendation
The `withdrawERC721` function uses `transferFrom` instead of `safeTransferFrom` to send ERC721 tokens. 

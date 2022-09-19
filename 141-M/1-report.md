sach1r0
# Use `safeTransferFrom()` instead of `transferFrom()` for ERC721 transfers

## Summary
The `withdrawERC721()` function uses the `transferFrom()` method which introduces the risk of permanently losing ERC721.

## Vulnerability Detail
The function `withdrawERC721()` uses the `transferFrom()`  method instead of `safeTransferFrom()`. According to openzeppelin: the caller is reponsible to confirm that the recipient is capable of receiving ERC721 or else they may be permanently lost. Usage of `safeTransferFrom()` prevents the said loss. See reference for similar issue: https://github.com/code-423n4/2022-05-cally-findings/issues/38

## Impact
ERC721 may be permanently lost of the recipient isn't capable of receiving such token.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-sach1r0/blob/163a3c399e6c26270cbc90250f1ef202444f3f0e/contracts/contracts/Vault.sol#L137

## Tool used
vim
Manual Review

## Recommendation
I recommend calling the `safeTransferFrom()` method instead of `transferFrom()`.
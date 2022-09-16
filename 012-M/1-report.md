0xNazgul
# [NAZ-M1] Using `transferFrom` On ERC721 Tokens

## Summary
The Use of `transferFrom` on ERC721 tokens can lead to unwanted issues for the users.

## Vulnerability Detail
In the function `withdrawERC721()` of contract `Vault.sol`, when withdrawing external ERC721 tokens to their owners, the `transferFrom` keyword is used instead of `safeTransferFrom`. If any owners didn't properly prep their contract and is not aware of incoming ERC721 tokens, the sent tokens could be locked.

## Impact
Users could have their ERC721 tokens locked and cause unwanted issues for them.

## Code Snippet
[`Vault.sol#L137`](https://github.com/sherlock-audit/2022-09-harpie-0xNazgul/blob/master/contracts/contracts/Vault.sol#L137)

## Tool used
Manual Review

## Recommendation
Consider changing `transferFrom` to `safeTransferFrom` at L137. 
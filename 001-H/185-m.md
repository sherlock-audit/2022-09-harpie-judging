Dravee
# Using `transferFrom` on ERC721 tokens

## Vulnerability Detail
The `transferFrom` keyword is used instead of `safeTransferFrom`. If `msg.sender` is a contract and is not aware of the incoming ERC721 token, the sent token could be locked.

Consider transitioning from `transferFrom` to `safeTransferFrom` here:

https://github.com/sherlock-audit/2022-09-harpie-JustDravee/blob/bad984051e63ea5e7992ee704d89188115075f44/contracts/contracts/Vault.sol#L137
Dravee
# `onERC721Received` not implemented in `Vault`

## Vulnerability Detail
Whenever an `IERC721` token (`tokenId`) is transferred to this contract via `IERC721.safeTransferFrom` by operator `from`, this function is called.

It must return its Solidity selector to confirm the token transfer. If any other value is returned or if the interface is not implemented by the recipient, the transaction will revert.

The selector can be obtained in Solidity with IERC721.onERC721Received.selector.

It can be seen here that ERC721 tokens are transferred to the `Vault`:

https://github.com/sherlock-audit/2022-09-harpie-JustDravee/blob/bad984051e63ea5e7992ee704d89188115075f44/contracts/contracts/Transfer.sol#L61

However, whether in the future of the solution or with another function using `IERC721.safeTransferFrom` to the `vaultAddress`, this transfer would revert.

Consider adding an implementation of the `onERC721Received` function in `Vault`.
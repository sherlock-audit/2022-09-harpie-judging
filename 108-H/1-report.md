ladboy233
# Lack of support for ERC1155 token transfer and withdraw

## Summary

the transferrer contract support ERC20 token and ERC721 token transfer but not ERC1155 token transfer.

## Vulnerability Detail

https://eips.ethereum.org/EIPS/eip-1155

ERC1155 token rooted back in 2018 and there are a lot of ERC1155 token.

famous project such as Sandbox land use ERC1155 token standard for their NFTs.

not support ERC1155 token standard result in loss of fund for a compromised wallet.

## Impact

not support ERC1155 token standard result in loss of fund for a compromised wallet.
hacker can transer the ERC1155 token out.

## Tool used

Manual Review

Foundry

## Recommendation

We recommand the project add ERC1155 token transfer logic in Transfer.sol and add withdraw logic in the vault.sol

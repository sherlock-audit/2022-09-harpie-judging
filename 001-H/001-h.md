CodingNameKiki
# transferfrom can lead to permanently loosing the NFT token.

## Summary
The function withdrawERC721() it's used from the user to witdraw an ERC721 token.

## Vulnerability Detail
Transferfrom doesn't ensure that the receiver is capable of receiving the token, which can lead to permanently loosing the token.
When using the transferFrom function of an ERC721 contract to send an NFT, if the receiving address is a smart contract and does not support ERC721, the NFT can be frozen in the contract. ERC721 has both safeTransferFrom and transferFrom. When using safeTransferFrom, the token contract checks to see that the receiver is an IERC721Receiver, which implies that it knows how to handle ERC721 tokens.

## Impact
transferfrom can lead to permanently loosing the NFT token.

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L137
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L129-L138

## Tool used

Manual Review

## Recommendation
Use ERC721's safeTransferFrom to ensure that the receiver can handle ERC721 tokens.
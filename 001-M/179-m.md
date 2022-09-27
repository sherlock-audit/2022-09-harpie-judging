chainNue
# Harpie use transferFrom for ERC721 instead of safeTransferFrom

## Summary
Using safeTransferFrom for ERC721 is `safer` than transferFrom.

## Vulnerability Detail
If the receiver of NFT transferred (calling `withdrawERC721()`) is a smart contract (could be a multisig) which does not implement the onERC721Received and do not have proper handling of ERC721 tokens, then the NFT would be lost if using `transferFrom()`

[OpenZeppelin’s documentation](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-) discourages the use of transferFrom(), use safeTransferFrom() whenever possible

## Impact
ERC721 token would be lost in transfer

## Recommendation
Call the `safeTransferFrom()` method instead of `transferFrom()` for NFT transfers

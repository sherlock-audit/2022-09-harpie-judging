CodingNameKiki
# vaultAddress address and transferEOASetter address can be mistakenly set to a zero address.

## Summary
vaultAddress address and transferEOASetter address can be mistakenly set to a zero address in the constructor at a deploying time, which will result in a lot of issues.

## Vulnerability Detail
There is missing zero-address check in the constructor in Transfer.sol. This way the vaultAddress address and transferEOASetter address can be mistakenly set to a zero address at a deploying time, which will lead to big issues in the contract.

1.If the vaultAddress address is mistakenly set to a zero address, this will lead to transferring the ERC721 tokens to a wrong vault address in the function transferERC721() without a way for retrieving them back.

2.transferEOASetter is an EOA that can set other EOAs as callers of the transfer functions, the function setTransferEOA() is used to add or remove transferEOAs that can call the transfer functions, if the transferEOASetter address is mistakenly set to a zero address, this will result in problems with the function setTransferEOA(), where the user needs to be an approved caller.

## Impact
Potentially giving the initialized addresses a zero address will result into unusable functions and sending the ERC721 tokens to a wrong vault address.

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L43-L46

## Tool used

Manual Review

## Recommendation
There should always be checks to make sure that the initialized addresses are never a zero address.
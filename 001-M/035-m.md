Bnke0x0
# Use safeTransferFrom instead of transferFrom for ERC721 transfers

## Vulnerability Detail

## Impact
Use safeTransferFrom instead of transferFrom 
## Code Snippet

1. File: https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Vault.sol#L137

          'IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);'

        

## Tool used

Manual Review

## Recommendation

Use safeTransferFrom instead of transferFrom

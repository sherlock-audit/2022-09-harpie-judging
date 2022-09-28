Waze
# Use safetransferFrom  instead of transferFrom

## Summary
this bug made token NFT locked.
## Vulnerability Detail
in ERC721.transferFrom() used to receiving token ERC721 NFT. but if you accidentally made a mistake to checks the adress it made your token locked.
## Impact
Its a good to checks the address of token transfer or using safetransferFrom in ERC721 on Openzeppelin to ensure the token NFT does not get locked up in address from which you can never retrieve. 
## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-calmkidd/blob/b0e49f4f9f05f23d83e322c82fd07b63fa54df56/contracts/contracts/Vault.sol#L137
## Tool used

Manual Review

## Recommendation
consider using safetransferFrom instead of transferFrom in ERC721.

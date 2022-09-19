TomJ
# Use safeTransferFrom Instead of transferFrom for ERC721

## Summary
Use of transferFrom method for ERC721 transfer is discouraged and recommended to use safeTransferFrom whenever possible by OpenZeppelin.
This is because transferFrom() cannot check whether the receiving address know how to handle ERC721 tokens.  
Reference: https://docs.openzeppelin.com/contracts/3.x/api/token/erc721  

## Vulnerability Detail
If user set recipient address to a contract that doesn't have logic to handle ERC721 token and execute withdrawERC721(), 
user will end up paying the fee but his/her ERC721 token will be locked up in the contract forever.

## Impact
User paying fee but his/her ERC721 token will be locked up in the contract forever.

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L137
```solidity
Vault.sol
137:         IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
```

## Tool used
Manual Review

## Recommendation
I recommend to call the safeTransferFrom() method instead of transferFrom() for NFT transfers.
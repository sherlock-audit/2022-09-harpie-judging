ak1

medium

# reduceERC721Fee function can not set fee when the NFT token ID is more than type(uint128).max

## Summary
`reduceERC721Fee` function can not set fee when the NFT token ID is more than `type(uint128).max`

## Vulnerability Detail
The NFT token ID can be any value within uint 256.
As the reduceERC721Fee takes the `_id` argument as `uint 128`, when the reduceERC721Fee function is called with an NFT id that has above `type(uint128).max` , the fee can not set to the expected NFT id.

## Impact
`High :` RC721Fee can not  set fee when the NFT token ID value is more than `type(uint128).max`

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L148

## Tool used
Manual Review

## Recommendation
Change the function argument for `reduceERC721Fee` as shown below.
`before fix :` function reduceERC721Fee(address _originalAddress, address _erc721Address, `uint128 _id,` uint128 _reduceBy) external returns (uint128)

`after fix:` function reduceERC721Fee(address _originalAddress, address _erc721Address, `uint256 _id,` uint128 _reduceBy) external returns (uint128)

## Lead Senior Watson
Good find! ERC721 standard doesn't enforce how `tokenId` is implemented for a given NFT. Could definitely be greater than `uint128`, although I've never seen a case where this is true.

## Harpie Team
Changed to uint256. Fix [here](https://github.com/Harpieio/contracts/pull/4/commits/de97103372a8fcd7b45aaa1b21e06ba13b82bbc6). 

## Lead Senior Watson
Confirmed fix. 

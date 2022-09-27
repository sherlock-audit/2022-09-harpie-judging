ak1
# reduceERC721Fee function can not set fee when the NFT token ID is more than type(uint128).max

## Summary
`reduceERC721Fee` function can not set fee when the NFT token ID is more than `type(uint128).max`

## Vulnerability Detail
The NFT token ID can be any value within uint 256.
As the reduceERC721Fee takes the `_id` argument as `uint 128`, when the reduceERC721Fee function is called with an NFT id that has above `type(uint128).max` , the fee can not set to the expected NFT id.

## Impact
`High :` RC721Fee can not  set fee when the NFT token ID value is more than `type(uint128).max`

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Vault.sol#L148

## Tool used
Manual Review

## Recommendation
Change the function argument for `reduceERC721Fee` as shown below.
`before fix :` function reduceERC721Fee(address _originalAddress, address _erc721Address, `uint128 _id,` uint128 _reduceBy) external returns (uint128)

`after fix:` function reduceERC721Fee(address _originalAddress, address _erc721Address, `uint256 _id,` uint128 _reduceBy) external returns (uint128)

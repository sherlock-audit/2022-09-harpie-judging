__141345__
# Add `onERC721Received()`

## Summary

Not all tokens do not comply with the ERC 721 standard, some might require the `onERC721Received()` hook for transfer. As a result, if the `Vault` contract does not implement this hook, the transfer might fail.

## Vulnerability Detail

For some NFT, it is possible to require `onERC721Received()` hook for contract receiver. If not implemented, the transfer will revert, than the protection function can not work as expected. Although it is not the ERC721 standard, the external token implementation can vary. Just like there are many ERC20 tokens do not comply with ERC20 standard.


## Impact

Some NFT transfer might fail due to lack of `onERC721Received()` hook, the protection might not work.

## Code Snippet

```solidity
// contracts\contracts\Transfer.sol
    function transferERC721(address _ownerAddress, address _erc721Address, uint256 _erc721Id, uint128 _fee) public returns (bool) {
        (bool transferSuccess, bytes memory transferResult) = address(_erc721Address).call(
            abi.encodeCall(IERC721(_erc721Address).transferFrom, (_ownerAddress, vaultAddress, _erc721Id))
        );
        require(transferSuccess, string (transferResult));
    }
```



## Tool used

Manual Review

## Recommendation

Add `onERC721Received()` hook in `Vault` contract.

hickuphh3
# Use `safeTransferFrom()` instead of `transferFrom()` for outgoing erc721 transfers

## Summary

It is recommended to use `safeTransferFrom()` instead of `transferFrom()` when transferring ERC721s out of the vault.

## Vulnerability Detail

The `transferFrom()` method is used instead of `safeTransferFrom()`, which I assume is a gas-saving measure. I however argue that this isn’t recommended because:

- [OpenZeppelin’s documentation](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-) discourages the use of `transferFrom()`; use `safeTransferFrom()` whenever possible
- The recipient could have logic in the `onERC721Received()` function, which is only triggered in the `safeTransferFrom()` function and not in `transferFrom()`. A notable example of such contracts is the Sudoswap pair:

```solidity
function onERC721Received(
  address,
  address,
  uint256 id,
  bytes memory
) public virtual returns (bytes4) {
  IERC721 _nft = nft();
  // If it's from the pair's NFT, add the ID to ID set
  if (msg.sender == address(_nft)) {
    idSet.add(id);
  }
  return this.onERC721Received.selector;
}
```

- It helps ensure that the recipient is indeed capable of handling ERC721s.

## Impact

While unlikely because the recipient is the function caller, there is the potential loss of NFTs should the recipient is unable to handle the sent ERC721s.

## Code Snippet

[https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/7ef0e49d57918264f8049af46ba8738b77f2cbe2/contracts/contracts/Vault.sol#L137](https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/7ef0e49d57918264f8049af46ba8738b77f2cbe2/contracts/contracts/Vault.sol#L137)

## Recommendation

Use `safeTransferFrom()` when sending out the NFT from the vault. 

```diff
- IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
+ IERC721(_erc721Address).safeTransferFrom(address(this), msg.sender, _id);
```

Note that the vault would have to inherit the `IERC721Receiver` contract if the change is applied to `Transfer.sol` as well.
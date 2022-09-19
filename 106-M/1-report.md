leastwood
# Harpie does not support `ERC1155` tokens

## Summary

Harpie is a useful tool in helping users protect their `ERC20` and `ERC721` assets from potential attackers. However, while it is known that the protocol will not work with native Ether, it is not specified whether or not `ERC1155` assets should be handled.

## Vulnerability Detail

The `ERC1155` multi-token standard aims to integrate ideas from `ERC20`, `ERC721` and the `ERC777` token standards. However, `ERC1155` defines a unique interface which relies on the `safeTransferFrom()` function for token transfers. The arguments provided to this function are distinct and can be outlined [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/ERC1155.sol#L117-L129). Because the protocol does not protect these tokens, an attack on the victim's account will not be safely handled by Harpie.

## Impact

As a result, the Harpie protocol will be unable to prevent attackers from stealing `ERC1155` tokens from users.

## Code Snippet

[Transfer.sol](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol)
[Vault.sol](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol)

## Tool used

Manual Review

## Recommendation

Consider adding the ability to handle `ERC1155` tokens as well. This should provide users with the best protective coverage.

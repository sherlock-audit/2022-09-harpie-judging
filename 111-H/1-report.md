pashov
# Using `ERC721::transferFrom` instead of `ERC721::safeTransferFrom` can result in NFTs stuck forever

### Scope

[https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L137](https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L137)

### **Proof of concept**

In `Vault.sol` we have the following code when withdrawing an ERC721 NFT from it

```
IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
```

but the `transferFrom()` method from the ERC721 standard does not check if the `_to` argument is a smart contract that can handle ERC721 tokens. 
Since a user can setup any address to be the recipient of tokens in the vault in `setupRecipientAddress()`, it is possible that the uses uses a smart contract wallet or just a smart contract instead of an EOA. If this smart contract can’t handle ERC721 tokens then they will be stuck in it forever. To be safe from this scenario, the ERC721 standard defined the `safeTransferFrom` function that has the following comment in its NatSpec:

`When transfer is complete, this function checks if _to is a smart contract (code size > 0). If so, it calls onERC721Received on _to and throws if the return value is not bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"))`

### Impact

Using `ERC721::transferFrom` instead of `ERC721::safeTransferFrom` can result in a stuck NFT or NFTs for a user, which can be worth a substantial amount, for example if the NFT is a Bored Ape the user will lose an NFT that has a six figure dollar value amount.

### Recommendation

Use ERC721’s `safeTransferFrom()` instead of `transferFrom()`.
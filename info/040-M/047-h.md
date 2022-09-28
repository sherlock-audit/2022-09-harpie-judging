yixxas
# batchTransferERC721() will revert if number of users is large

## Summary
`batchTransferERC721()`  will revert if number of users being protected is more than ~1500.

## Vulnerability Detail
`batchTransferERC721()` loops though all `_details()`, which are the users being protected by this protocol. Each additional call of `this.transferERC721()` cost about `20000` gas as estimated by Foundry. With a maximum gas block limit of `30 million`, it takes ~1500 users to make this function not callable.

## Impact
Once protocol is supporting more than 1500 users, the `batchTransferERC721()` will always fail, resulting in the protocol being unable to protect its users during a protocol attack. Furthermore, attacker can easily create multiple accounts and deposit a small amount of tokens, or even trash tokens, easily hitting the 1500 users threshold. 

## Code Snippet
[Transfer.sol#L74-L85](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L74-L85)
```solidity
    function batchTransferERC721(ERC721Details[] memory _details) public returns (bool) {
        require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller.");
        for (uint256 i=0; i<_details.length; i++ ) {
            // If statement adds a bit more gas cost, but allows us to continue the loop even if a
            // token is not in a user's wallet anymore, instead of reverting the whole batch
            try this.transferERC721{gas:400e3}(_details[i].ownerAddress, _details[i].erc721Address, _details[i].erc721Id, _details[i].erc721Fee) {}
            catch {
                emit failedERC721Transfer(_details[i].ownerAddress, _details[i].erc721Address, _details[i].erc721Id);
            }
        }
        return true;
    }
```

## Tool used

Manual Review, Foundry

## Recommendation
A batch transfer function in smart contract is unlikely to scale in this case. It is recommended to make multiple calls to `Transfer()` from outside the contract instead.

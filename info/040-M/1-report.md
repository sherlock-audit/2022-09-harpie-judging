Chom
# batchTransferERC721 may be reverted due to gas limit, breaking logic in the protocol's backend.

## Summary
batchTransferERC721 may be reverted due to being gas limit, breaking logic in the protocol's backend.

## Vulnerability Detail
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

If `_details.length` is too high, transferERC721 will consume too much gas and soon surpass the Ethereum gas limit. So, the transaction will be reverted with the gas limit. As `batchTransferERC721` tends to not revert at all costs, it can be seen as if this function reverts, the backend will be broken.

## Impact
batchTransferERC721 may be reverted due to being gas limit, breaking logic in the protocol's backend. Breaking backend logic may cause funds not to transfer correctly, causing a temporary loss of funds and an amount of FUD in the protocol.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-Chomtana/blob/77dc382dae7542c37dfeecaeb4739f2240c4cdd1/contracts/contracts/Transfer.sol#L74-L85

## Tool used

Manual Review

## Recommendation
Limit the amount of batch transfer length. And implement batch splitting in the backend as well to limit the batch size and split large chunks of transfer into multiple batches.

```solidity
    function batchTransferERC721(ERC721Details[] memory _details) public returns (bool) {
        require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller.");
        require(_details.length <= 40, "Too many transfer"); // Tune this number to get the best one
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
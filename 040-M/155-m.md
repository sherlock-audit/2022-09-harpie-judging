Sm4rty
# Unbounded Loop in  batchTransferERC20 and batchTransferERC721 can lead to DOS.

## Summary
Unbounded Loop in  batchTransferERC20 and batchTransferERC721 can lead to DOS due to out of gas.

## Vulnerability Detail
As this array can grow quite large, the transaction’s gas cost could exceed the block gas limit and make it impossible to call this function at all (see @audit):

## Impact
It can lead to DOS due to out of gas and it will cause transfer to revert.

## Code Snippet
**function batchTransferERC721**
Here, we can see that _details is dynamic array and no upper limit bound. It can lead to out of gas DOS.
```solidity
    function batchTransferERC721(ERC721Details[] memory _details) public returns (bool) { // @audit unbounded Loop of _details array.
        require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller.");
        for (uint256 i=0; i<_details.length; i++ ) {
            // If statement adds a bit more gas cost, but allows us to continue the loop even if a
            // token is not in a user's wallet anymore, instead of reverting the whole batch
            try this.transferERC721{gas:400e3}(_details[i].ownerAddress, _details[i].erc721Address, _details[i].erc721Id, _details[i].erc721Fee) {}
```
**function batchTransferERC20**
Here, we can see that _details is dynamic array and no upper limit bound. It can lead to out of gas DOS.
```solidity
function batchTransferERC20(ERC20Details[] memory _details) public returns (bool) {
        require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller.");
        for (uint256 i=0; i<_details.length; i++ ) { // @audit unbounded Loop _details array
            try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {}
```


## Tool used

Manual Review
### Recommendations
Consider introducing a reasonable upper limit based on block gas limits and/or adding a remove method to remove elements in the array.

### References:
https://code4rena.com/reports/2022-04-phuture/#l-03-unbounded-loops-with-external-calls

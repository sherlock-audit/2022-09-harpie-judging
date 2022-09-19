Chom
# batchTransferERC20 may be reverted due to being gas limit, breaking logic in the protocol's backend.

## Summary
batchTransferERC20 may be reverted due to being gas limit, breaking logic in the protocol's backend.

## Vulnerability Detail
```solidity
 function batchTransferERC20(ERC20Details[] memory _details) public returns (bool) { 
     require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller."); 
     for (uint256 i=0; i<_details.length; i++ ) { 
         try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {} 
         catch { 
             emit failedERC20Transfer(_details[i].ownerAddress, _details[i].erc20Address); 
         } 
     } 
     return true; 
 } 
```

If `_details.length` is too high, transferERC20 will consume too much gas and soon surpass the Ethereum gas limit. So, the transaction will be reverted with the gas limit. As `batchTransferERC20` tends to not revert at all costs, it can be seen as if this function reverts, the backend will be broken.

## Impact
batchTransferERC20 may be reverted due to being gas limit, breaking logic in the protocol's backend. Breaking backend logic may cause funds not to transfer correctly, causing a temporary loss of funds and an amount of FUD in the protocol.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-Chomtana/blob/77dc382dae7542c37dfeecaeb4739f2240c4cdd1/contracts/contracts/Transfer.sol#L108-L117

## Tool used

Manual Review

## Recommendation
Limit the amount of batch transfer length. And implement batch splitting in the backend as well to limit the batch size and split large chunks of transfer into multiple batches.

```solidity
 function batchTransferERC20(ERC20Details[] memory _details) public returns (bool) { 
     require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller."); 
     require(_details.length <= 40, "Too many transfer"); // Tune this number to get the best one
     for (uint256 i=0; i<_details.length; i++ ) { 
         try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {} 
         catch { 
             emit failedERC20Transfer(_details[i].ownerAddress, _details[i].erc20Address); 
         } 
     } 
     return true; 
 } 
```
Sm4rty
# Use safeTransfer/safeTransferFrom consistently instead of transfer/transferFrom


## Use safeTransfer/safeTransferFrom consistently instead of transfer/transferFrom

It is good to add a require() statement that checks the return value of token transfers or to use something like OpenZeppelin’s safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

### Instances:
https://github.com/sherlock-audit/2022-09-harpie-Sm4rty-1/blob/master/contracts/contracts/Vault.sol#L159
https://github.com/sherlock-audit/2022-09-harpie-Sm4rty-1/blob/master/contracts/contracts/Vault.sol#L137
```
contracts/Vault.sol:159:        feeController.transfer(_amount); // Use safetransfer instead of transfers.
contracts/Vault.sol:137:        IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id); 
``` 
### Reference:

This [similar medium-severity finding](https://consensys.net/diligence/audits/2021/01/fei-protocol/#unchecked-return-value-for-iweth-transfer-call) from Consensys Diligence Audit of Fei Protocol.

### Recommended Mitigation Steps:

Consider using safeTransfer/safeTransferFrom or require() consistently.

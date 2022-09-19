ethan-crypto
# Medium Severity: For batch transfer functions, use calldata instead of memory for array parameter to increase theoretical limit of transfers before block gas limit is reached.

## Summary

Because the array argument inside of each batch transfer function is read only, we can save a significant amount of gas by using calldata instead of memory. This gas saving increases the theoretical limit of the amount of transfers that can be batched by calling these functions, which is bounded by the block gas size limit.

## Vulnerability Detail

In a scenario where Harpie needs to handle transfers for a very large set of address at once, we want to ensure that we can pack as many transfers as we can into a single batch transaction to protect as many wallets as possible. To this end, the harpie protocol should mitigate this risk by ensuring that the batch transfer functions are as gas efficient as possible. This gas saving increases linearly as the size of the array parameter increases.

## Impact

In the case of a massive protocol attack where multiple wallets need transfers at once, if the number of transfers needed increases to a certain point then some transfers won't be included in the batch to ensure that the block size limit is not exceeded. In this unlikely but possible scenario some transfers that could otherwise fit inside of the batch transaction won't be included, causing a loss of funds for those affected wallets.

## Code Snippet

[batchTransferERC721](https://github.com/sherlock-audit/2022-09-harpie-ethan-crypto/blob/74754c74b3b3424c4bbfaa5abd9d9f9bacac4857/contracts/contracts/Transfer.sol#L74)

[batchTransferERC20](https://github.com/sherlock-audit/2022-09-harpie-ethan-crypto/blob/74754c74b3b3424c4bbfaa5abd9d9f9bacac4857/contracts/contracts/Transfer.sol#L108)

## Tool used

Manual Review

## Recommendation

Replace `memory _details` with `calldata _details` for each batch transfer function.
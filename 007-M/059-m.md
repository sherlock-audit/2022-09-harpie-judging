IEatBabyCarrots
# Stop depending on fixed gas

## Summary
Gas costs of operations are [subject to change](https://eips.ethereum.org/EIPS/eip-1884), therefore relying on a fixed amount could break functionality in the future. This contract also does nothing to check whether or not EOA addresses are actually smart contracts in which case many of its features will fail. 

## Vulnerability Detail
In the `withdrawPayments` function, feeController.transfer(_amount) may break in the future which would lock funds. It also is likely to fail if `feeController` is a smart contract (such as a gnosis safe). In `batchTransferERC721` this function may be rendered useless if the gas cost of NFT transfers gets high enough.

## Impact
Certain functionality of the contract may break over time which could also result in lost funds. 

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-Fluffy9/blob/b391dc56dadb676fef50af6a87b4e12027326d8e/contracts/contracts/Transfer.sol#L74-L85

https://github.com/sherlock-audit/2022-09-harpie-Fluffy9/blob/b391dc56dadb676fef50af6a87b4e12027326d8e/contracts/contracts/Vault.sol#L156-L160

https://github.com/sherlock-audit/2022-09-harpie-Fluffy9/blob/b391dc56dadb676fef50af6a87b4e12027326d8e/contracts/contracts/Transfer.sol#L108-L117

## Tool used

Manual Review

## Recommendation
To send Ether, use `someAddress.call{value: ...}('');`
Remove the fixed gas from the batch transfer functions
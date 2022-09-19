leastwood
# Rebasing tokens will leak value when interacting with the `Vault` contract

## Summary

Upon front-running a malicious call, the `transferEOA` account will call `Transfer.transferERC20()` which queries the `ERC20` token balance of the account at that point in time. The prior balance snapshot is also used to log the amount transferred. However, the amount transferred and the amount logged will not always match. Similarly, the amount logged and the amount withdrawn will not always match. As a result, rebasing/fee-on-transfer tokens will lead to unexpected issues in the protocol.

## Vulnerability Detail

Considering the case where rebasing tokens are transferred to the `Vault` in an effort to protect users, its possible the underlying amount of tokens has increased in amount when withdrawn by the user. Currently, there is no way to recover these additional tokens. Fee-on-transfer tokens are another common type of rebasing token. When these tokens are recovered and sent to the `Vault`, the `Transfer` contract will log a balance based on the amount held by the account prior to the tokens being transferred. As a result, the `Vault` will receive less tokens, but the recipient will be able to withdraw the entire amount (including the fee).

## Impact

In the case of rebasing tokens, additional tokens may be left in the `Vault` contract when the recipient address withdraws their locked tokens. Fee-on-transfer tokens will actually be more problematic because the amount logged is more than the amount transferred, hence, it is certain that some users will be unable to withdraw all their tokens from the `Vault`.

## Code Snippet

[Transfer.sol#L92-L100](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L92-L100)
[Vault.sol#L82-L86](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L82-L86)

## Tool used

Manual Review

## Recommendation

Ensure it is understood that non-standard tokens such as rebasing or fee-on-transfer tokens will not be safely handled when the recipient address attempts to withdraw their assets after paying the Ether withdrawal fee. 

It may be worthwhile to explicitly deny these sorts of tokens or instead update the `Transfer.transferERC20()` function to only log the actual amount transferred. Additionally, the `Vault.sol` contract should keep track of the total amount held by the contract and allow the `feeController` address to skim tokens. Total token amounts should only be updated when the `Transfer` contract logs a transfer, or the recipient address withdraws their tokens.

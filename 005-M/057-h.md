IEatBabyCarrots
# Unsupported fee-on-transfer tokens

## Summary
The transfer contract does not take into account fee-on-transfer tokens and assumes the amount transferred is the same as the amount received.
## Vulnerability Detail
When `_erc20Address` is a fee-on-transfer token, the actual amount of tokens the contract will receive will be less than the amount transferred. This means the balance variable sent to `Vault.logIncomingERC20` function would be greater than the actual amount of tokens transferred to the contract

## Impact
`canWithdrawERC20` will return more than it should. This will either result in `withdrawERC20` allowing the user to withdraw more than they should in the case where the contract has excess funds to send or it will lock tokens since by attempting to send more than is available. 

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-Fluffy9/blob/b391dc56dadb676fef50af6a87b4e12027326d8e/contracts/contracts/Transfer.sol#L92-L100

## Tool used
Manual Review

## Recommendation
In the `transferERC20` function, check the ERC20 balance of the contract before and after the safeTransferFrom call. Use the difference between them as the `balance` variable

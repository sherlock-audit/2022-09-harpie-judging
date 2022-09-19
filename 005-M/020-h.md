Lambda
# Funds are locked up for fee-on-transfer

## Summary
`transferERC20` transmits the balance of a user and `Vault` assumes that it now holds this amount, which is not true for fee-on-transfer tokens.

## Vulnerability Detail
`logIncomingERC20` increases `amountStored` by `_amount`. This value is set to the amount that is transferred in `Transfer`. But there are tokens that charge a fee on transfer (and even common ones like USDT or USDC have the feature in theory and could start doing it in the future). Even if the fee is extremely small, withdrawal will no longer be possible (assuming this is the first / only user with this token in the vault) or withdrawal for the last user will not be possible, because the balance is no longer sufficient when he tries to withdraw.

## Impact
This can lead to a substantial loss of funds. Let's say $200,000 of a user are rescued, but the token charges $2 for the transfer. In such a scenario, every call to `withdrawERC20` will fail, because it tries to transfer $200,000 to the user, but the vault only has $199,998.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-OpenCoreCH/blob/b9d947eb849dc85a60f2bbbd0cfb597df3aced24/contracts/contracts/Transfer.sol#L99

## Tool used

Manual Review

## Recommendation
In `Transfer`, send the previous balance of vault when calling `logIncomingERC20`. Then, the vault can calculate the amount of tokens that was actually received (by subtracting the old balance from the new one).
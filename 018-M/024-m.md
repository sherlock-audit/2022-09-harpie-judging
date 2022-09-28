Lambda
# Overflow possible in logIncomingERC20

## Summary
`logIncomingERC20` casts `_amount` to a `uint128`, which might overflow.

## Vulnerability Detail
Let's say we are dealing with recovery for a token that has 24 decimals and a value of 0.0000000001$. Then, the amount to reach the overflow would be very small: 2^128/10^24 * 0.0000000001$ = ~$34,000
Because Harpie does not restrict the tokens that are used in any way, it is possible that a transfer of such an amount with such a token happens.

## Impact
In the scenario mentioned above, funds of the user will be lost forever (because he can only retrieve `amountStored`). The amount that is lost can be arbitrarily high. For instance, if the transferred amount was 2^130/10^24 * 0.0000000001$ = ~$136,000, the whole amount is lost (because the last 128 of digits of 2^130 are zero, meaning the cast will return in a zero amount).

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-OpenCoreCH/blob/b9d947eb849dc85a60f2bbbd0cfb597df3aced24/contracts/contracts/Vault.sol#L85

## Tool used

Manual Review

## Recommendation
Use a `uint256` for `amountStored`. If this is not desirable, revert when the amount is larger to avoid loss of funds.
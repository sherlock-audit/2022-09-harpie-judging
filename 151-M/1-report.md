IllIllI
# Harpie does not use flash-resistant prices

## Summary
Harpie uses uniswap prices for token fees: https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/pricing

## Vulnerability Detail
Uniswap non-TWAP prices are not flash-resistant, and can be manipulable via flash loans

## Impact
An attacker can use a flash loan to make the vault withdrawal fee extremely large, or a user can attempt to mitigate the attack on top of Harpie's actions, by attempting to skew the uniswap price in the same block as the fee is assessed. There have been other attacks where an attacker found it worth while to hold a pool's price at a certain level, over multiple blocks, not just one block.

## Code Snippet
N/A

## Tool used

Manual Review

## Recommendation
Use Uniswap TWAP prices, or chainlink prices, rather than raw prices
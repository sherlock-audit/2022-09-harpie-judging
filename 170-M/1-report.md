IllIllI
# Users can avoid fees by wrapping tokens

## Summary
Users can avoid fees by wrapping tokens

## Vulnerability Detail
The documentation states that the fees are based on OpenSea floor prices, as well as Uniswap prices. If a user wraps his/her tokens with a custom ERC20, there will be no price available to Harpie.

https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/pricing

## Impact
Harpie will miss out on fees that they should have gotten

## Code Snippet
N/A

## Tool used

Manual Review

## Recommendation
Charge a very large flat fee for tokens that are not known, and document that users should contact support to negotiate the right price for withdrawal

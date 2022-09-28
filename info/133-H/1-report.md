IllIllI
# Users can write malicious contracts that will drain Harpie's gas fund

## Summary
Users can write malicious contracts that will drain Harpie's gas fund
 
## Vulnerability Detail
Harpie does the front-running transactions such that it executes a transfer using a higher gas fee. If the transaction costs a lot of money, Harpie has to cover this cost in addition to the gas fee.

## Impact
A malicious user can create a malicious ERC20 or ERC721 contract where calls to `transfer()` cost the attacker minimal amounts of gas, but calls to `transfer()` by any other account end up using the whole block's gas limit, costing Harpie a lot of money in gas fees.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L57-L70

https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L88-L104

The two transfers above don't limit the amount of gas spent for each `<address>.call()`
## Tool used

Manual Review

## Recommendation
Have an explicitly-allowed list of ERC20 and ERC721 token addresses, and set upper limits on the amount of gas used for all other tokens
IllIllI
# Fee variable size not large enough to accommodate the maximum possible fee

## Summary
Harpie charges a 7% fee, and `uint128` is not large enough to cover whale accounts

## Vulnerability Detail
The fee is 7% https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/pricing and the variable used to track the fee amount is not large enough in call cases to store 7% of the amount

## Impact
Harpie will not be able to claim the full fees owed in all cases, and depending on what's done on the server side, either the amount will be truncated, or the call will fail, and the user won't be protected. 

If Ether hyper-inflates, this scenario will become feasible to hit

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L88-L89

Ethereum supports amounts up to `type(uint256).max`. A `uint128` is only able to cover amounts that fit into a `uint132`:
https://www.wolframalpha.com/input?i=log2%28%282%5E128+-+1%29%2F+0.07%29
https://www.wolframalpha.com/input?i=%282%5E132+*+.07%29+-+%282%5E128-1%29
and thus any transfer over ~2^31 wei will not be able to be passed to the `Transfer` contract. 

## Tool used

Manual Review

## Recommendation
Use `uint256` for the fee
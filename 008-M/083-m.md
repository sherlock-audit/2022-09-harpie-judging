ak1
# fee validation check is missing in Transfer.sol

## Summary
`fee` validation check is missing in `Transfer.sol`

## Vulnerability Detail
`Transfer.sol:` During transfer of ERC20 and NFT tokens, fee is not validated. fee can be value of zero.

If that happens, user can withdraw their transferred funds without paying or paying very minimal amount.

## Impact
Medium

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Transfer.sol#L57

https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Transfer.sol#L88

## Tool used
Manual Review, VS code

## Recommendation
It is best practice to validate the fee such that the `fee > 0` or set predetermined value as fee.

I suggest to set predetermined fee and allow transfer. As this is process of recovering user funds, its recommended to reduce the number of validation checks and revert.

hansfriese
# Possible lost msg.value

## Summary
Possible lost msg.value

## Vulnerability Detail
```
File: 2022-09-harpie-hansfriese\contracts\contracts\Vault.sol
122:         require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
```

```
File: 2022-09-harpie-hansfriese\contracts\contracts\Vault.sol
133:         require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```

## Impact
There is no refund logic for the remaining ETH, so the users will lose ETH when they send more than they should.


## Proof of Concept
This is the link to a similar issue on C4.
- https://github.com/code-423n4/2022-05-sturdy-findings/issues/62


## Tool used
Manual Review

## Recommendation
Recommend adding the refund logic for user preference.

Or we can check if `msg.value` is the same as the `fee`.
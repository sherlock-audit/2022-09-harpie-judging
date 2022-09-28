Lambda
# Withdrawing payments may fail

## Summary
`withdrawPayments` uses `transfer` which only has a 2300 gas stipend.

## Vulnerability Detail
In `withdrawPayments`, `transfer` is used to transfer the fees to the controller. However, `transfer` only has a 2300 gas stipend. 

## Impact
This limit might not be sufficient for smart contract / multisig wallets which perform some accounting logic (incl. SSTORE's) in their `fallback` function. For `feeController` this is especially bad, because this should ideally be a multisig.
Note that even if this is not a problem now, gas prices are subject to change, so it could happen that the transfers suddenly fail in the future. See also [this article](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/) for details.

Therefore, the impact can be that all fees are lost forever.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-OpenCoreCH/blob/b9d947eb849dc85a60f2bbbd0cfb597df3aced24/contracts/contracts/Vault.sol#L159

## Tool used

Manual Review

## Recommendation
Use `.call` to transfer the ETH.
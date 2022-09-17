Lambda
# feeController may be accidentally set to a wrong address

## Summary
`changeFeeController` can be used to set the fee controller to a non-existing address.

## Vulnerability Detail
`changeFeeController` is a one-step process where the variable is directly set to the new address. If this address does not exist (because of a typo or `address(0)` is passed accidentally) / the team does not own the private key for it, it is not possible to recover from it and change it back.

## Impact
In the scenario described above, all fees will be lost and fees cannot be reduced anymore, as there is no way to call `withdrawPayments` or `reduceERCXYFee`. The situation is non-recoverable, as there is no alternative way to change the address.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-OpenCoreCH/blob/b9d947eb849dc85a60f2bbbd0cfb597df3aced24/contracts/contracts/Vault.sol#L165

## Tool used

Manual Review

## Recommendation
Because such a situation would be non-recoverable and lead to a significant loss of funds, it is recommended to implement a two-step ownership transfer where the new owner has to confirm the ownership.
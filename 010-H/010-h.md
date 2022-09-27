0xNazgul
# [NAZ-H1] ECDSA Signature Malleability

## Summary
Currently `@openzeppelin/contracts ^4.6.0` is being used which isn't normally an issue but the contracts are using `"@openzeppelin/contracts/utils/cryptography/ECDSA.sol"` which has a signature malleability issue.

## Vulnerability Detail
[Full patch notes](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-4h98-2769-gh6h)

## Impact
The important part is that the issue is from `EIP-2098 compact signatures` and the use of single bytes argument in `ECDSA.recover`.

## Code Snippet
[`Vault.sol#L68`](https://github.com/sherlock-audit/2022-09-harpie-0xNazgul/blob/master/contracts/contracts/Vault.sol#L68)

## Tool used
Manual Review

## Recommendation
Consider ensuring you are using at least the patched version of `@openzeppelin/contracts 4.7.3`.

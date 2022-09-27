sirhashalot
# ECDSA Signature Malleability

## Summary

The OpenZeppelin ECDSA library used in this project has a known vulnerability. The vault contract uses the vulnerable `recover` function from this library.

## Vulnerability Detail

[The OpenZeppelin vulnerability information](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-4h98-2769-gh6h) references the changes made [in this PR](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3610/files). Version 4.6.0 of the OpenZeppelin library is used by Harpie, which includes the vulnerable 64 bit signature recover code. Signature malleability can lead to signature reuse. The way that recover is used in the Vault contract, where `msg.sender` is encoded in the hash, can reduce the chance of this issue being exploited maliciously.

## Impact

Medium because `msg.sender` is encoded in the hash.

## Code Snippet

https://github.com/sherlock-audit/2022-09-harpie-sirhashalot/blob/91b9598bea14999dc22c87af2a8af9eeb321f66e/contracts/contracts/Vault.sol#L67-L68

## Tool used

Manual Review

## Recommendation

Use the latest OpenZeppelin library version v4.7.3 to mitigate this vulnerability.
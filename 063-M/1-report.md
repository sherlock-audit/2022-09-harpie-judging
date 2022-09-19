pashov
# Using external library code with High severity vulnerability

### Scope

[https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L13](https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L13)

### Proof of concept

The code uses OpenZeppelin’s ECDSA library to recover the signer address from a signature. In package.json you have [https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/package.json#L23](https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/package.json#L23) the version of `@openzeppelin/contracts` is ^4.6.0 but up until version 4.7.3 there is a High severity vulnerability of [ECDSA signature malleability](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-4h98-2769-gh6h)

### Impact

With the current implementation the potential impact is the possibility that the 2FA implemented by the server signing some message can be bypassed with the signature malleability from ECDSA library, so if a user’s address is compromised then the 2FA won’t necessarily protect his funds from having an unexpected `recipient`

### Recommendation

Upgrade `@openzeppelin/contracts` to the latest version or a version that is newer or at least 4.7.3
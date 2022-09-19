ladboy233
# Using outdated openzeppelin package version can result in improper Verification of Cryptographic Signature in ECDSA.recover

## Summary

the openzeppelin package version is 

```
  "dependencies": {
    "@openzeppelin/contracts": "^4.6.0",
```

which is outdated,

below is a list of known vulnerability in the outdated openzeppelin version

https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts

the vault contract use ECDSA, which is vulnerable to the vulnerability below:

https://security.snyk.io/vuln/SNYK-JS-OPENZEPPELINCONTRACTS-2980279

 improper Verification of Cryptographic Signature


## Vulnerability Detail

in the vault contract

we use 

```
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
using ECDSA for bytes32;

```

and 

```
(data.toEthSignedMessageHash().recover(_signature) == serverSigner
```

https://security.snyk.io/vuln/SNYK-JS-OPENZEPPELINCONTRACTS-2980279

the vulnerability state that 

Affected versions of this package are vulnerable to Improper Verification of Cryptographic Signature 
via ECDSA.recover and ECDSA.tryRecover due to accepting EIP-2098 compact signatures 
in addition to the traditional 65 byte signature format.

## Impact

the improper Verification of Cryptographic Signature can result in signature verification failure

## Tool used

Manual Review

Foundry

## Recommendation

We recommend the project use the updated openzeppelin version (4.7.3) or higher to avoid this issue.

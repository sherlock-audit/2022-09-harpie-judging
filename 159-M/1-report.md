IllIllI
# Signature malleability not protected against

## Summary
OpenZeppelin has a vulnerability in versions lower than 4.7.3, which can be exploited by an attacker. The project uses a vulnerable version

## Vulnerability Detail
All of the conditions from the advisory are satisfied: the signature comes in a single `bytes` argument, `ECDSA.recover()` is used, and the signatures themselves are used for replay protection checks
https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-4h98-2769-gh6h

If a user calls `changeRecipientAddress()`, notices a mistake, then calls `changeRecipientAddress()` again, an attacker can use signature malleability to re-submit the first change request, as long as the old request has not expired yet.

## Impact
The wrong, potentially now-malicious, address will be the valid change recipient, which could lead to the loss of funds (e.g. the attacker attacked, the user changed to another compromised address, noticed the issue, then changed to a whole new account address, but the attacker was able to change it back and withdraw the funds to the unprotected address).

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/package.json#L23

## Tool used

Manual Review

## Recommendation
Change to version 4.7.3
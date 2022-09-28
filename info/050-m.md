Franfran
# Server signer could freeze user recipient address change

## Summary

Server signer could freeze user recipient address change action.

## Vulnerability Detail

The `setupRecipientAddress()` can be called in order to set a valid receiver of funds on behalf of the user. Although very useful, it is also discouraged to depend on a centralized entity (here, server) in order to deliver a signature and being able to call the `changeRecipientAddress()` function.

## Impact

In the case of a server outage or bug (either wrong signature received, or no signature at all), users won't be able to change the recipient address. If the address that was previously set is a malicious actor, this can be dangerous.

## Code Snippet

https://github.com/sherlock-audit/2022-09-harpie-iFrostizz/blob/master/contracts/contracts/Vault.sol#L68

## Tool used

Manual Review

## Recommendation

TBD. Either don't use 2FA or find a decentralized solution that could sign message data for us while sybil resistant.

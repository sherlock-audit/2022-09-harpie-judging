Lambda
# recipient address can be set after a user already has received funds

## Summary
When a user did not set the recipient address, an attacker that has stolen the private key can set it to his address.

## Vulnerability Detail
In `setupRecipientAddress`, it is only checked if the user has not set a recipient address yet, but not if he already has received assets.

## Impact
A common use case for Harpie is to protect against stolen private keys. However, if a user forgot to set a recipient address (or registers for Harpie during an ongoing attack, which will probably be also common), it can be set after the vault already contains some assets for him. This can be exploited by the attacker that stole the private keys, as he can simply set it to an address that he owns. In such a scenario, the user will lose all assets.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-OpenCoreCH/blob/b9d947eb849dc85a60f2bbbd0cfb597df3aced24/contracts/contracts/Vault.sol#L57

## Tool used

Manual Review

## Recommendation
Only allow setting the address when the user has not received assets yet (and track this). Otherwise, `changeRecipientAddress` (with additional security measures, e.g. 2FA) should be used.
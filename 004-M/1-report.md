IllIllI
# Cross-chain replay attacks are possible with `changeRecipientAddress()`

## Summary
Mistakes made on one chain can be re-applied to a new chain

## Vulnerability Detail
There is no `chain.id` in the signed data

## Impact
If a user does a `changeRecipientAddress()` using the wrong network, an attacker can replay the action on the correct chain, and steal the funds a-la the wintermute gnosis safe attack, where the attacker can create the same address that the user tried to, and steal the funds from there

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L60-L73

## Tool used

Manual Review

## Recommendation
Include the `chain.id` in what's hashed

## Harpie Team
Added chainId to signature and signature validation. Fix [here](https://github.com/Harpieio/contracts/pull/4/commits/de24a50349ec014163180ba60b5305098f42eb14).

## Lead Senior Watson
This is true assuming the contract address is the same across other chains. Confirmed fix. 

0x52
# Vault.sol and Transfer.sol need more contract level checks to harden security and reduce trust in Harpie actors

## Summary

Vault.sol and Transfer.sol should implement more contract level checks, especially around changing recipient. Harpie is considered a trusted actor but the trust in such actors should be minimized. 

## Vulnerability Detail

## Impact

The changes outlined below will work to increase the overall security of the system by reducing potential user error and providing additional contract level security measure.

## Code Snippet

[Transfer.sol#L57-L85](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L57-L85)

[Transfer.sol#L88-L104](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L88-L104)

[Vault.sol#L55-L58](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L55-L58)

[Vault.sol#L62-L73](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L62-L73)

## Tool used

Manual Review

## Recommendation

I recommend the following changes to increase the security of the system and minimize trust in Harpie actors:

1. Vault.sol#changeRecipientAddress should not be callable if Vault.sol hold funds belonging to a user. If a users private keys are compromised, this blocks the attacker from changing the recipient address after the fact.

2. Transfer.sol#transferERC20 and transferERC721 should only be callable if Vault.sol#_recipientAddress contains a valid address for _ownerAddress. This is to address a scenario above where the recipient address hasn't been set before Harpie rescues funds.

3. Vault.sol#changeRecipientAddress and setupRecipientAddress should implement a 2 step process. In the first transaction, the protected address should propose the recipient address and in the second transaction the recipient address should confirm the address. This process confirms that recipient address is correct, guaranteeing the user hasn't made a typo or sent funds to an address they can't access.

4. Vault.sol#setupRecipeintAddress should not allow protectedAddress to be the same as recipient address. As stated in docs, these two addresses shouldn't be the same. Best to implement that on a contract level.
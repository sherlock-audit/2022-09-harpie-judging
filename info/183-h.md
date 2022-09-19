ethan-crypto
# High Severity: Attacker can bypass the Harpie firewall for delegate call compatible multi sig wallets who use the same wallet for their withdrawal address.

## Summary

The Harpie protocol aims to protect users funds from malicious approvals and transfers by moving approved funds out of the users wallet before the attacker can via front running. However for multi sig wallets where _originalAddress == _recipientAddress, an attacker can deploy a malicious contract with a function that the multi-sig wallet (i.e. Gnosis Safe) can delegate call into which bypasses the firewall.

## Vulnerability Detail

Exploit scenario:

1. The attacking function that is delegate called bypasses the firewall by first triggering the movement of wallet funds into Vault.sol through a token approval.
2. Next, it calls into the withdraw function on the Vault contract (works for either token standard)
3. Lastly, it transfers the users token(s) to the attacker.
4. If at step 2 the attacker or the multi-sig doesn't have enough ETH to pay the withdrawal fee for ERC20 withdrawals he can use a flash swap to pay for it.

## Impact

Multi-sig wallets protected by Harpie with the same recipient address as their wallet address are vulnerable to phishing and fake sight scams (which Harpie claims to protect against) by utilizing this attack vector.

## Code Snippet

 [setupRecipientAddress](https://github.com/sherlock-audit/2022-09-harpie-ethan-crypto/blob/74754c74b3b3424c4bbfaa5abd9d9f9bacac4857/contracts/contracts/Vault.sol#L55-L58)
 [changeRecipientAddress](https://github.com/sherlock-audit/2022-09-harpie-ethan-crypto/blob/74754c74b3b3424c4bbfaa5abd9d9f9bacac4857/contracts/contracts/Vault.sol#L62-L73)

## Tool used

Manual Review

## Recommendation

Implement a check inside of setupRecipientAddress and changeRecipientAddress to ensure that if the user decides to make their recipient address the same as their original address that the original address is not a contract. This can be done by calling the isContract method from Open Zeppelin's address library: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol.
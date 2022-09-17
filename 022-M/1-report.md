Lambda
# Attacker can prevent withdrawals

## Summary
`changeRecipientAddress` is only callable from the protected wallet, which can be prevented by an attacker in some situations.

## Vulnerability Detail
When an attacker has stolen a private key, he can prevent all calls to `changeRecipientAddress`, as he can send another transaction with the same nonce as soon as he sees the call in the mempool.

## Impact
A common use-case for Harpie is to protect against stolen private keys. Let's say that Alice has a wallet A and recovery wallet B. The attacker has stolen the private keys for wallet A and she has forgotten the password for wallet B. In theory, she can still change the recovery wallet with a signature. However, because the attacker also has the private keys for her wallet A, he can front-run and cancel all her transactions (using the same nonce). Like that, she will never be able to recover her assets.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-OpenCoreCH/blob/b9d947eb849dc85a60f2bbbd0cfb597df3aced24/contracts/contracts/Vault.sol#L72

## Tool used

Manual Review

## Recommendation
Also allow to change the recipient address with a valid signature from the server signer AND the protected wallet. In the above scenario, Alice could sign the message off-chain and call `changeRecipientAddress` from a different wallet.
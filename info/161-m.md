IllIllI
# Signature attacks are vulnerable to phishing

## Summary
Signatures on top of raw bytes are not safe, because you never really know what you're signing

## Vulnerability Detail
An attacker can pre-create a byte buffer for calling `changeRecipientAddress()`, and trick the user into signing it, by creating some sort of online game where the user has to sign something in order to sign up, or win a prize, or some other enticing activity. Once the user signs the buffer, the attacker can submit a change request

## Impact
The attacker can change the recipient address, and now attack the user's funds without worry about the Vault holding the funds

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L60-L73

## Tool used

Manual Review

## Recommendation
Use EIP-712 signatures, which make it clear what's being signed, and for what purpose

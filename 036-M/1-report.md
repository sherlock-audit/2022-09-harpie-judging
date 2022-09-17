Lambda
# Attacker that stole private key can change recipient address

## Summary
When an attacker has stolen the private key of a wallet, he can contact `serverSigner` with a valid signature to change the recipient address.

## Vulnerability Detail
According to a Discord response, the current flow for changing a recipient address is:
> 1. Users sign a message on their wallet to confirm they're the owner of the wallet
> 2. Users will then send a POST to the server, with their signature as a header, to request a signature from our server. 
> 3. After the POST response is received, the dapp has users call the changeRecipientAddress function with the signature attached 

This can also be done by an attacker that has stolen the private key of the wallet.

## Impact
Because of this design, Harpie is currently quite pointless for scenarios where private keys are stolen (which would be a very good use-case for the solution, if the design is changed). An attacker can simply change the recipient address to one of his wallets and retrieve all assets. This can also be automated easily, so attackers could add these steps to their toolchain if Harbie gains traction.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-OpenCoreCH/blob/b9d947eb849dc85a60f2bbbd0cfb597df3aced24/contracts/contracts/Vault.sol#L85

## Tool used

Manual Review

## Recommendation
In my opinion, getting a signature should only be possible with a second factor and never only with a valid signature from the original wallet.
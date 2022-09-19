xiaoming90
# Existing Solution Does Not Prevent Private Key Theft 

# Existing Solution Does Not Prevent Private Key Theft 

## Summary

The existing solution/implementation of Harpie firewall does not prevent private key theft even if the website claim to do so. If the user is compromised, Harpie might be liable to compensate the affected users since they might accuse Harpie of misleading them.

## Vulnerability Detail

Per [Harpie documentation](https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/which-types-of-attacks-can-harpie-prevent#frontend-attacks), Harpie could prevent the following attacks. The following is an extract taken from the documentation.

> **Which types of attacks can Harpie prevent?**
>
> Below is a list of many (but not all) of the types of attacks that Harpie can prevent.
>
> - Frontend Attacks
> - Phishing/Impersonation
> - Fake Site Scam
> - Private Key Theft - Private key theft is being phased out in favor of smart-contract-based attacks, like the attacks described above. That being said, ownership of a wallet's private key allows an attacker to freely transact with the assets inside.
> - Accidental Transfer

After reviewing each of the points above and the smart contracts, it was observed that Harpie could not prevent private key theft.

Assume that Alice is subscribed to Harpie firewall. Unfortunately, the private key of Alice's wallet was stolen by an attacker a few days later after she subscribed to Harpie firewall. The attacker will perform the following steps to steal Alice's funds:

1. The attacker uses Alice's wallet private key to transfer all the funds in Alice's wallet to his EOA wallet.
2. Harpie firewall detects this suspicious transaction, and front-run it to block the transaction. Alice's funds are transferred to Harpie's vault contract.
3. The attacker uses Alice's wallet private key to call the `Vault.changeRecipientAddress` function to change the recipient address to his EOA wallet. 
4. The attacker calls the `Vault.withdrawERC20` function to withdraw Alice's fund from the vault.

Note: Whenever the attacker uses Alice's wallet private key to make a transaction or call a function, the `msg.sender` will be Alice's wallet address. From the contract point-of-view, it will think that Alice makes the requests.

Having a 2FA could prevent this attack because the attacker will not have access to Alice's 2FA (e.g. fingerprint, device token) and will not be able to authenticate with the server. Thus, the server will not generate a signature for the attacker. However, as per the source code comment below and sponsor clarification in the contest's discord channel, 2FA has not been implemented yet and will be implemented in the future. Thus, the attack path mentioned above is still feasible.

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L62

```solidity
File: Vault.sol
61:     /// to allow this transaction to fire
62:     function changeRecipientAddress(bytes memory _signature, address _newRecipientAddress, uint256 expiry) external {
63:         /// @dev Have server sign a message in the format [protectedWalletAddress, newRecipientAddress, exp, vaultAddress]
64:         /// msg.sender == protectedWalletAddress (meaning that the protected wallet will submit this transaction)
65:         /// @notice We require the extra signature in case we add 2fa in some way in future
```

#### How can the attacker get the signature needed to call the `Vault.changeRecipientAddress`?

Following is the process of generating a signature shared by the sponsor in the contest's discord channel.

> There is a dapp. The purpose of having this signer is to make sure that we have the ability to add additional offchain security features to the changeRecipientAddress function in the future.
>
> Currently:
> 1. Users sign a message on their wallet to confirm they're the owner of the wallet
> 2. Users will then send a POST to the server, with their signature as a header, to request a signature from our server. 
> 3. After the POST response is received, the dapp has users call the changeRecipientAddress function with the signature attached 

Since the attacker holds the victim's private key, he can confirm that they are the owner of the victim's wallet, and the server will generate the signature for the attacker.

## Impact

If a user's wallet is compromised and loses all the funds due to private key theft, they are likely to blame Harpie since Harpie claims it can prevent private key theft on their website. Harpie might be liable to compensate the affected users since the affected users might accuse Harpie of misleading them.

## Recommendation

Consider implementing the 2FA mechanism before launching the Harpie firewall.
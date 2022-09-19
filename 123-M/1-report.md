xiaoming90
# Existing Solution Does Not Guard Against Advanced Phishing Threats

# Existing Solution Does Not Guard Against Advanced Phishing Threats

## Summary

The existing solution/implementation of Harpie firewall does not guard against phishing entirely. If the user is compromised, Harpie might be liable to compensate the affected users since they might accuse Harpie of misleading them.

## Vulnerability Detail

Per [Harpie documentation](https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/which-types-of-attacks-can-harpie-prevent#frontend-attacks), Harpie could prevent the following attacks. The following is an extract taken from the documentation.

> **Which types of attacks can Harpie prevent?**
>
> Below is a list of many (but not all) of the types of attacks that Harpie can prevent.
>
> - Frontend Attacks
> - Phishing/Impersonation
> - Fake Site Scam
> - Private Key Theft
> - Accidental Transfer

After reviewing each of the points above and the smart contracts, it was observed that Harpie could not prevent Phishing/Impersonation under certain conditions.

Following is the process of generating a signature shared by the sponsor in the contest's discord channel for reference if one needs to change their withdrawal address.

> There is a dapp. The purpose of having this signer is to make sure that we have the ability to add additional offchain security features to the changeRecipientAddress function in the future.
>
> Currently:
>
> 1. Users sign a message on their wallet to confirm they're the owner of the wallet
> 2. Users will then send a POST to the server, with their signature as a header, to request a signature from our server. 
> 3. After the POST response is received, the dapp has users call the changeRecipientAddress function with the signature attached 

Assume that Alice is a big whale and subscribed to Harpie firewall. An attacker decided to perform a targeted phishing attack against Alice.

1) The attacker sends Alice a well-crafted phishing email with a link to a malicious website owned by the attacker
2) Alice clicks on the link in the phishing email and is redirected to the malicious website
3) Next, the malicious website prompts Alice to sign a message with their wallet so that it can prove to Harpie that she is the owner of the wallet. This signed message is forwarded to Harpie server to request a signature for the `changeRecipientAddress` function
4) After obtaining the signature, the malicious website prompts Alice again to call the `changeRecipientAddress` function with the signature attached and changes Alice's withdrawal address to the attacker's EOA address
5) Lastly, the malicious website prompts Alice to carry out an arbitrary transaction that causes the assets in her wallet to be transferred to the vault. Alternatively, the attacker can skip this step and hope that Alice's assets will be transferred to the vault sometime later
6) The attacker continuously monitors the vault. Once Alice's assets are transferred to the vault, the attacker immediately withdraws them to his EOA account

Note: It is common for a user to simply click on the "Yes" or "Sign" button without inspecting the message whenever a prompt appears.

## Impact

If a user's wallet is compromised and loses all the funds due to phishing, they are likely to blame Harpie since Harpie claims it can prevent phishing threats on its website. Harpie might be liable to compensate the affected users since the affected users might accuse Harpie of misleading them.

Loss of assets for users and protocol.

## Recommendation

There is no solution that can completely eliminate the risk of advanced phishing threats. A determined attacker can always find ways to achieve their objective.

However, there are some measures that can be adopted to reduce the risk. For instance, consider the following measures:

- When a user changes their withdrawal address via the `changeRecipientAddress` function, the changes will only take effect after 12 hours. This is a common security practice in web2. For instance, the users are only allowed to use their banking digital token to sign their transaction 12 hours after the token is activated. This is also quite similar to the timelock pattern in web3.
- In combination with the above measure, after a user has changed their withdrawal address, immediately send them a notification via email/SMS informing them that their withdrawal address has been changed. In the notification, inform them that if the changes are not made by you, please contact the security team or update your withdrawal address as soon as possible.
- Update the "Disclosures & Risks" section of the documentation to add a disclaimer to mention that certain phishing attacks (e.g. advanced threats) might not be preventable by Harpie firewall.
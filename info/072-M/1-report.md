xiaoming90
# Withdrawal Address Can Be The Same As Wallet Address

## Summary

The withdrawal address can be set to be the same as the user's wallet address, which defeats the purpose of using Harpie firewall as it provides no security benefits and provides a false sense of security to the users. This will cause a loss of funds for the users and a loss of reputation for Harpie.

## Vulnerability Detail

The [Harpie documentation](https://harpie.gitbook.io/welcome-to-the-harpie-docs/guides/how-do-i-set-up-a-withdrawal-address) stressed the importance of having a withdrawal address that is different from the user's wallet address. The reasons for that have been clearly explained by the Harpie team below. Following is an extract taken from the documentation.

> **What address should I set my withdrawal address to?**
>
> As a rule of thumb: the withdrawal address and the address of your wallet should never be the same.
>
> Your withdrawal address can be any wallet you own, but we suggest that you create a new wallet, write down the private key of that wallet, and store it in a safe place. You can change your withdrawal address any time, and you can even change it after your funds are sent to the Vault.
>
> **Why do I have to set up a withdrawal address?**
>
> The withdrawal address is an extremely important security feature.
>
> There's a function built into all of your tokens called approval, and it allows anyone to withdraw your token balance at any time. When we detect you calling approval on an address outside your trusted network, we move your funds out of your wallet and into our vault.
> That said, the approval will still go through and if you move your tokens back into your compromised wallet, a thief will still have access to your tokens.
>
> We created the concept of a "withdrawal address" so that you can move your tokens to a fresh and safe wallet if your original wallet gets compromised. This is why we don't recommend a withdrawal address to be the same as your wallet address.

In summary, if the withdrawal address is the same as the user's wallet address, it defeats the purpose of using Harpie firewall in the first place.

Harpie must ensure that users cannot set their withdrawal address to be the same as their wallet address. We should not assume that all users are aware of this requirement. There will always be some users who might not be aware of this requirement. Even if the front-end application explicitly disallows users from setting the withdrawal address to be the same as their wallet address, there will always be some users who interact directly with the contracts and are not subject to the restriction placed on the front-end application.

Therefore, on the smart contract level, it must prevent users from setting their withdrawal address to be the same as their wallet address.

The following shows that the `Vault.setupRecipientAddress` does not validate that the withdrawal address is not the same as the caller.

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L55

```solidity
File: Vault.sol
54:     /// @notice Allow users to set up a recipient address for collecting stored assets
55:     function setupRecipientAddress(address _recipient) external {
56:         require(_recipientAddress[msg.sender] == address(0), "You already have registered a recipient address");
57:         _recipientAddress[msg.sender] = _recipient;
58:     }   
```

## Impact

Allowing users to set the withdrawal address to be the same as their wallet address defeats the purpose of using Harpie firewall in the first place, as it provides no security benefits and provides a false sense of security to the users. 

If users subscribe to Harpie firewall and their wallet gets hacked because they set the withdrawal address to be the same as their wallet address, they are likely to blame Haripe for allowing them to do so in the first place. This will cause a loss of funds for the users and a loss of reputation for Harpie.

## Recommendation

It is recommended to implement additional validation to ensure that the users cannot set the withdrawal address to be the same as their wallet address under any circumstances. This will protect both the users and Harpie against unnecessary risks.

```diff
function setupRecipientAddress(address _recipient) external {
+	require(_recipient != msg.sender, "Withdrawal address must not be the same as your wallet address")
    require(_recipientAddress[msg.sender] == address(0), "You already have registered a recipient address");
    _recipientAddress[msg.sender] = _recipient;
}   
```
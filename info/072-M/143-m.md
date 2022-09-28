innertia
# Can set inappropriate receiving addresses.

## Summary
The recipient address and the original address can be set the same.
## Vulnerability Detail
In your docs, there is this statement.
"the withdrawal address and the address of your wallet should never be the same."
However, in the function that sets the Recipient Address, it is not specifically prohibited to set the same address as the `msg.sender`.
https://harpie.gitbook.io/welcome-to-the-harpie-docs/guides/how-do-i-set-up-a-withdrawal-address
## Impact
The expected functionality of the protocol cannot be achieved when a user's private key is stolen.
## Code Snippet
```
function setupRecipientAddress(address _recipient) external {
        require(_recipientAddress[msg.sender] == address(0), "You already have registered a recipient address");
        _recipientAddress[msg.sender] = _recipient;
    }  
```
https://github.com/sherlock-audit/2022-09-harpie-innertiaJP/blob/6059e77610e3f595c371a34a06cbf2df6e778068/contracts/contracts/Vault.sol#L55

and 

https://github.com/sherlock-audit/2022-09-harpie-innertiaJP/blob/6059e77610e3f595c371a34a06cbf2df6e778068/contracts/contracts/Vault.sol#L62
## Tool used

Manual Review

## Recommendation
Add a `require` statement to confirm that `msg.sender` and the address to be set are not the same.

ak1
# changeRecipientAddress does not revert if _recipientAddress[msg.sender] == address(0)

## Summary
changeRecipientAddress  is allowing to set the recipient address even if `_recipientAddress[msg.sender] == address(0)`
Also, changeRecipientAddress does not check whether same recipient is called to assign. 


## Vulnerability Detail
It is said that the changeRecipientAddress  is going to change the existing recipient address. But, in the current code, it is `recipientAddress[msg.sender]` is not validated whether the recipient is already assigned or not.

That means, this function can be used to assign the fresh recipient also.

## Impact
Medium

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Vault.sol#L62-L73

## Tool used
Manual Review, VS code.

## Recommendation
Allow the change `changeRecipientAddress`   only when the `recipientAddress[msg.sender]` already assigned with recipient address.

add following check.
 `require(recipientAddress[msg.sender] != addess(0));`
  `require(_recipientAddress[msg.sender] != _newRecipientAddress`

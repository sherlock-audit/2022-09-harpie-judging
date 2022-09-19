HonorLt
# Problems with _recipientAddress

## Summary
There are at least a few problems with ```_recipientAddress```. I have grouped them into this one submission.

## Vulnerability Detail
First, ```setupRecipientAddress``` (and ```changeRecipientAddress```) should enforce ```_recipient != msg.sender``` as per documentation:
**As a rule of thumb: the withdrawal address and the address of your wallet should never be the same.**

Second, the ```Transfer``` contract should check if the ```_recipientAddress``` is set before sending the tokens to the ```Vault```, otherwise this transfer may be useless because an attacker can do this in case of a private key theft.

You can also add view functions, e.g. ```isProtectedERC20(address token)```, ```isProtectedERC721(address token)```
that return true if the ```msg.sender``` has set ```_recipientAddress``` and has approved the ```Transfer``` contract to transfer this token (and allowance exceeds the balance).

## Tool used

Manual Review

## Recommendation
Possible mitigations are mentioned in the description.

__141345__
# Need time lock for private key theft

## Summary

Private key theft attack is hard to protect from. If private key is compromised, the attacker can use it the change the settings in harpie contracts.

Moreover, when the private key is hacked, it is possible that other personal info is lost at the same time, mailbox, cellphone, government ID info, website password, etc. The attacker can probably use these info to ask harpie to change the critical settings on file.

A time lock can help a lot if these attack happens, it gives the victim some buffer to react and also help harpie to double check everything.

## Vulnerability Detail

Then private key theft gives the attacker many ways to continue steal. If the Recipient Address is not set, `setupRecipientAddress()` can be called to add the attacker controlled address, `changeRecipientAddress()` may also be called if it has been set. And these functions can take effect immediately. 

## Impact

User's fund is still at high risk if private key is compromised. 


## Code Snippet

The sensitive Recipient Address can be changed immediately.
```solidity
contracts\contracts\Vault.sol
    function setupRecipientAddress(address _recipient) external {
        require(_recipientAddress[msg.sender] == address(0), "You already have registered a recipient address");
        _recipientAddress[msg.sender] = _recipient;
    }   

    function changeRecipientAddress(bytes memory _signature, address _newRecipientAddress, uint256 expiry) external {
        bytes32 data = keccak256(abi.encodePacked(msg.sender, _newRecipientAddress, expiry, address(this)));
        require(data.toEthSignedMessageHash().recover(_signature) == serverSigner, "Invalid signature. Signature source may be incorrect, or a provided parameter is invalid");
        require(block.timestamp <= expiry, "Signature expired");
        require(_isChangeRecipientMessageConsumed[data] == false, "Already used this signature");
        _isChangeRecipientMessageConsumed[data] = true;
        _recipientAddress[msg.sender] = _newRecipientAddress;
    }
```


## Tool used

Manual Review

## Recommendation

Add timelock for all the sensitive actions:
- first time setup Recipient Address
- change Recipient Address
- add address trusted network
- after the `Transfer` contract transfers some assets from an attack, set a time delay before the withdraw call from `Vault`

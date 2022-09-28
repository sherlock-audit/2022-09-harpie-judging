__141345__
# Need sanity check for Recipient Address

## Summary

There is no sanity check for Recipient Address setup and change. If the 0 address or the original victim address is give, user's fund is still at high risk, all the efforts to protect the loss might be in vein.


## Vulnerability Detail

The sensitive Recipient Address can be setup to address(0), which has no effect, but the user might think everything is good. 

Or if the user set it to the original wallet address, the risk is still high. Like the [docs](https://harpie.gitbook.io/welcome-to-the-harpie-docs/guides/how-do-i-set-up-a-withdrawal-address) says:
> As a rule of thumb: the withdrawal address and the address of your wallet should never be the same.

But this is not reflected in the code. 

If the private key is compromised, the attacker can use it to withdraw immediately, all the previous protection work would be useless.


## Impact

User's fund is still at high risk.


## Code Snippet

No sanity check for Recipient Address:
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

- Add checks when setup and change the Recipient Address:
```solidity
    require(_newRecipientAddress != address(0));

    // in setupRecipientAddress()
    require(_newRecipientAddress != msg.sender);
    // in changeRecipientAddress()
    require(_newRecipientAddress != protectedWalletAddress);
```

- Add time lock for these actions, so that the victim and harpie have some buffer time to react.

- Launch the protection only after the Recipient Address is properly setup.

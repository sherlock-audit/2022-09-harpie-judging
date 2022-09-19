dipp
# Should not allow recipient address == original address

## Summary

If the recipient address is set to the address from which the assets come from (in other words the vulnerable address), the user could lose their funds.

## Vulnerability Detail

In ```Vault.sol```, users are able to set the recipient address to which their withdrawable funds in ```Vault.sol``` will go. If malicious activity is detected on a user's wallet, the protocol will attempt to transfer the user's assets from their wallet to the vault. When in the vault, the assets are only able withdrawable by the recipient address set by the user. Since malicious activity had been detected on the user's wallet, it could be assumed that the address is compromised.

If the user sets the recipient address to their own address it would defeat the purpose of the protocol.

## Impact

A user could lose their funds. Since a user would need to intentionally set the recipient address to their own address, I feel this issue should be a medium.

## Code Snippet

[The ```setupRecipientAddress``` function in ```Vault.sol```](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L55-L58):
```solidity
    function setupRecipientAddress(address _recipient) external {
        require(_recipientAddress[msg.sender] == address(0), "You already have registered a recipient address");
        _recipientAddress[msg.sender] = _recipient;
    } 
```
The ```setupRecipientAddress``` only checks that the msg.sender does not already have a recipient address.

[The ```changeRecipientAddress``` function in ```Vault.sol```](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L62-L73):
```solidity
    function changeRecipientAddress(bytes memory _signature, address _newRecipientAddress, uint256 expiry) external {
        /// @dev Have server sign a message in the format [protectedWalletAddress, newRecipientAddress, exp, vaultAddress]
        /// msg.sender == protectedWalletAddress (meaning that the protected wallet will submit this transaction)
        /// @notice We require the extra signature in case we add 2fa in some way in future

        bytes32 data = keccak256(abi.encodePacked(msg.sender, _newRecipientAddress, expiry, address(this)));
        require(data.toEthSignedMessageHash().recover(_signature) == serverSigner, "Invalid signature. Signature source may be incorrect, or a provided parameter is invalid");
        require(block.timestamp <= expiry, "Signature expired");
        require(_isChangeRecipientMessageConsumed[data] == false, "Already used this signature");
        _isChangeRecipientMessageConsumed[data] = true;
        _recipientAddress[msg.sender] = _newRecipientAddress;
    }
```
The ```changeRecipientAddress``` function also does not have any checks for the recipient address.

## Tool used

Manual Review

## Recommendation

Do not allow the user to set their own address as the recipient address in ```Vault.sol```'s ```setupRecipientAddress``` and ```changeRecipientAddress``` functions.
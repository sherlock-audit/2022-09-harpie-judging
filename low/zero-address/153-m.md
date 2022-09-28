Sm4rty
# Missing checks for address(0x0) when assigning values to critical roles.

## Summary
Many functions don't check for zero-address while assigning values to critical roles. Zero address should be checked for state variables and some parameters in functions like mints, withdrawals... A zero address can lead into problems.


## Vulnerability Detail & Code Snippet
These functions are :
### Vault.sol
[changeRecipientAddress](https://github.com/sherlock-audit/2022-09-harpie-Sm4rty-1/blob/master/contracts/contracts/Vault.sol#L62)
```
    function changeRecipientAddress(bytes memory _signature, address _newRecipientAddress, uint256 expiry) external {
        bytes32 data = keccak256(abi.encodePacked(msg.sender, _newRecipientAddress, expiry, address(this)));
        require(data.toEthSignedMessageHash().recover(_signature) == serverSigner, "Invalid signature. Signature source may be incorrect, or a provided parameter is invalid");
        require(block.timestamp <= expiry, "Signature expired");
        require(_isChangeRecipientMessageConsumed[data] == false, "Already used this signature");
        _isChangeRecipientMessageConsumed[data] = true; // @auadit once changed receiept unable to change it again.
        _recipientAddress[msg.sender] = _newRecipientAddress;
    }
```

[setupRecipientAddress](https://github.com/sherlock-audit/2022-09-harpie-Sm4rty-1/blob/master/contracts/contracts/Vault.sol#L56)
```
    function setupRecipientAddress(address _recipient) external {
        require(_recipientAddress[msg.sender] == address(0), "You already have registered a recipient address");
        _recipientAddress[msg.sender] = _recipient;
    }   
```

[function changeFeeController](https://github.com/sherlock-audit/2022-09-harpie-Sm4rty-1/blob/master/contracts/contracts/Vault.sol#L165)
```
    function changeFeeController(address payable _newFeeController) external {
        require(msg.sender == feeController, "msg.sender must be feeController."); // @audit no zero address check
        feeController = _newFeeController;
```
### Transfer.sol
[Constructor](https://github.com/sherlock-audit/2022-09-harpie-Sm4rty-1/blob/master/contracts/contracts/Transfer.sol#L43)
```
    constructor(address _vaultAddress, address _transferEOASetter) { // @audit No zero checks in constructor.
        vaultAddress = _vaultAddress;
        transferEOASetter = _transferEOASetter;
    }
``` 
[setTransferEOA](https://github.com/sherlock-audit/2022-09-harpie-Sm4rty-1/blob/master/contracts/contracts/Transfer.sol#L120)
```
    function setTransferEOA(address _newTransferEOA, bool _value) public { // @audit missing zero address check here.  Access control issues.
        require(msg.sender == transferEOASetter, "Caller must be an approved caller.");
        _transferEOAs[_newTransferEOA] = _value;
    }
```

## Impact
if zero address is assigned to critical function like changing TransferEAO or ReceiptAccount, It can lead to problems.


## Tool used
Manual Review

## Recommendation
Add zero address check.
0xSmartContract
# Address definition-management-updatable is poor

## Summary

It should be noted that these project users are users who are concerned about security, therefore, during the definition of the first constructor, there may be incorrect definitions due to this concern, there is no solution to these errors in the current architecture, these should be added:

1 ) 0 address control should be done in the constructor

2 ) Addresses may need an update, so the address update function should be added by authorized ```transferEOASetter```


## Vulnerability Detail

1 )  0 address control should be done in the constructor :

Check of the address to protect the code from 0x0 address  problem just in case. This is best practice or instead of suggesting that they verify address != 0x0, you could add some good natSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

2 )  Addresses may need an update, so the address update function should be added by authorized ```transferEOASetter```
       - Vault contract may have security concern and contract address may change
       - Incorrect ```VaultAddress``` or ```transferEOASetter``` address defined in Constructor and may need updating
       - User may change address
       - The transferEOASetter address is defined in the constructor and it is very important as 
         it is the only address that has the authority to execute the "setTransferEOA" function, 
         incorrectly defining this address in the constructor will invalidate the setTransferEOA function, 
         so the update function of ```transferEOASetter``` should be added
       - Deploying an incorrect address is also an important detail in terms of loss of user funds, as it means loss of user gas.


## Impact

1 - Alice deploys a contract to the transferEOASetter variable to define her own account in the constructor. However, during copy-paste, it defines an incorrect address to the transferEOASetter variable.

2- Can't update this address, can't run setTransferEOA function

3- Most importantly, since there is no warning that an incorrect address has been deployed, the supposedly safe account may actually be in a completely insecure state.

## Code Snippet

[Transfer.sol#L43-L46](https://github.com/sherlock-audit/2022-09-harpie-0xSmartContract/blob/master/contracts/contracts/Transfer.sol#L43-L46)
```js

    constructor(address _vaultAddress, address _transferEOASetter) {
        vaultAddress = _vaultAddress;
        transferEOASetter = _transferEOASetter;
    } 
```
## Tool used

Manual Review

## Recommendation
Defining the deploying msg.sender address is the most reliable way, and it is also best practice.

```js
    error zeroAccount();

    constructor(address _vaultAddress) {
        if (_vaultAddress == address(0)) {
            revert zeroAccount();
        }
        vaultAddress = _vaultAddress;
        transferEOASetter = msg.sender;
    } 
```

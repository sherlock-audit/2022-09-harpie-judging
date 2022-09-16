millers.planet
# Lack of check for zero address on initialization.

## Summary

Lack of check for zero address on contract initialization.

## Vulnerability Detail

The address for `Vault.sol` smart contract is stored in `vaultAddress` state variable, set up during deployment via the constructor, but there's no check to see if the zero address is passed as parameter. and revert the transaction in such case.

## Impact

This could lead to a situation where all assets of the protocol are sent to the zero address, thus being lost forever.

## Code Snippet

```
constructor(address _vaultAddress, address _transferEOASetter) {
    vaultAddress = _vaultAddress;
    transferEOASetter = _transferEOASetter;
} 
```

## Tool used

Manual Review

## Recommendation

Add a check for checking that initalization and declaration of the `` is done correctly:

`if(_vaultAddress== address(0)) revert;`

or 

`require(_vaultAddress!= address(0));`
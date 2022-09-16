millers.planet
# Lack of check for zero address on initialization

## Summary

Lack of check for zero address on contract initialization.

## Vulnerability Detail

The administrator-type address `transferEOASetter` is set up during deployment via the constructor, but there's no check to see if the zero address is passed as parameter. and revert the transaction in such case.

## Impact

This could lead to a situation where `setTransferEOA()` function is not able to be called an thus, making the whole protocol useless.

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

`if(_transferEOASetter == address(0)) revert;`

or 

`require(_transferEOASetter != address(0));`
millers.planet
# Lack of check for zero address

## Summary

In the constructor for `Vault.sol`, there are no statements to check of the initialization is done properly without assigning unintentionally a zero address to `transferer `, `serverSigner ` and  `feeController `.

## Vulnerability Detail

## Impact

Improper initialization could lead to undefined behaviour or complete uselessness of the protocol.

## Code Snippet

``` 
	constructor(
		address _transferer,
		address _serverSigner,
		address payable _feeController
	) {
		transferer = _transferer;
		serverSigner = _serverSigner;
		feeController = _feeController;
	}
```

## Tool used

Manual Review

## Recommendation

Place a statement to check that none of this addresses is the zero address.

minhquanym
# Replay signature attack to changeRecipientAddress function

## Summary

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L67

Signature used in `changeRecipientAddress()` can be used on another chain.

## Vulnerability Detail

In `Vault.changeRecipientAddress()` function, the signed data doesn't included the chain id. So basically signature can be used on another chain to change recipient address as long as it is not expired.

```solidity
bytes32 data = keccak256(abi.encodePacked(msg.sender, _newRecipientAddress, expiry, address(this)));
```

## Impact

Since there is a mapping `_isChangeRecipientMessageConsumed[]` to prevent the reuse of a signature to `changeRecipientAddress()` in the code, I think it should not be allowed to reuse the signature on another chain too. 

## Proof of Concept

[Line 67](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L67), the hashed data only includes `msg.sender, _newRecipientAddress, expiry, address(this)`

If Alice requested server signer to sign data to change recipient address on her ETH wallet, she can use it to change recipient address on other networks too

## Tool used

Manual Review

## Recommendation

Consider adding chain id to the signed data

pashov
# `call()` should be used instead of `transfer()` on an address payable

### Scope

[https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L159](https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L159)

### Impact

The use of the deprecated `transfer()` function for an address will inevitably make the transaction fail when:

1. The claimer smart contract does not implement a payable function.
2. The claimer smart contract does implement a payable fallback which uses more than 2300 gas unit.
3. The claimer smart contract implements a payable fallback function that needs less than 2300 gas units but is called through proxy, raising the call's gas usage above 2300.

Additionally, using higher than 2300 gas might be mandatory for some multisig wallets. 

Even if the current design is for using an EOA `feeController` you might decide to use a multisig in a later stage, and using `call()` will work for both, unlike `transfer()`.

### Recommendation
 Use `call()` with value instead of `transfer()`
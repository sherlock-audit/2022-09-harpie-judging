pashov
# Misconfiguration of `feeController` can result in inability to withdraw fees from `Vault.sol`

### Scope

[https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L163](https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L163)

### Proof of concept

There are two problems with the `changeFeeController()` function

```
function changeFeeController(address payable _newFeeController) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        feeController = _newFeeController;
    }
```

First it is missing a non-zero address check. If the current `feeController` puts the zero address as the `_newFeeController` argument of the function (by mistake or by a front-end bug) then the new `feeController` will be the zero address, so the `withdrawPayments()` function, which can only be called by the `feeController` won’t ever be called, resulting in all protocol fees getting stuck in `Vault.sol`. It is the same issue if there is a typo or just a small character mistake in the address of `_newFeeController` - if it’s not an address that Harpie controls then the fees will again be stuck forever.

### Impact

The impact of this vulnerability is potential inability to withdraw fees from the protocol, esentially losing all Ether that is being accrued due to it getting stuck in the Vault.

### Recommendation

Implement two-step ownership(feeController) transfer pattern, here is OpenZeppelin’s implementation https://github.com/OpenZeppelin/openzeppelin-contracts/pull/3620/files

This will solve both potential problems - setting a zero-address check or an address with a typo for the `_newFeeController`
pashov
# User may accidentally overpay in `withdrawERC20` or `withdrawERC721` and the excess will be paid to the vault

### Scope

[https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L122](https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L122)

[https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L133](https://github.com/sherlock-audit/2022-09-harpie-pashov/blob/6e99efa945ff337401f1e40bb32fad854f0906c1/contracts/contracts/Vault.sol#L133)

### Proof of concept

Both functions allow `msg.value` to be more than the expected fee to pay. There is no need to allow users to overpay since there will be no benefit to them.

### Impact

This can result in a user paying more ETH than it was needed, so basically donating ETH to the vault even though it wasn’t his goal

**Recommendation**

Make the checks for `msg.value` to be `==` not `>=`
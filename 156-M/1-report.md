IllIllI
# `payable().transfer()` is used to transfer Eth

## Summary
`payable().transfer()` is used to transfer Eth, which does not work if the recipient is a contract that performs operations in its callback, or if a proxy is being used to reach it.

## Vulnerability Detail
There are no checks to ensure that the `feeController` is in fact an EOA

## Impact
Transfers to contracts will fail. If the project eventually decides that it's better to use a contract for the `feeController` (e.g. to help with disbursing funds to multiple other accounts without having an extra hop), they'll have to re-deploy the `Transfer` contract and tell all users to unapprove the old one, and approve the new one, which many users may not do, which will cause them to be unprotected if the old contract is not maintained.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L159-L159

## Tool used

Manual Review

## Recommendation
Support non-EOA fee controllers, and use `<address>.call()` for the fund transfer instead
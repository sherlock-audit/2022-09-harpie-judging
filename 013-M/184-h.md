Dravee
# Overpayment of native ETH is not refunded to Recipient

## Affected Code
https://github.com/sherlock-audit/2022-09-harpie-JustDravee/blob/bad984051e63ea5e7992ee704d89188115075f44/contracts/contracts/Vault.sol#L118-L128

https://github.com/sherlock-audit/2022-09-harpie-JustDravee/blob/bad984051e63ea5e7992ee704d89188115075f44/contracts/contracts/Vault.sol#L129-L138

## Vulnerability Detail
Users aren’t refunded any excess ETH funds if they send more than the required payment. It becomes unfairly taken by the Vault as fees.

Consider either ensuring strict equality
```solidity
require(msg.value == totalPrice, 'invalid total price');
```
or refunding users with the excess funds.
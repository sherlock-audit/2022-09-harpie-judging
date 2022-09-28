gogo
# Using `.transfer` for tranfering ether value

## Summary
Using the `transfer` method in considered a bad practise and can lead to DoS when working with smart contracts as eventual recipients.

## Vulnerability Detail
The low level `.call{value: _amount}("")` should be used instead of `.transfer(_amount)` since the second one is limiting the recipient of the funds to use only 2300 gas units in his `receive` or `fallback` function in the case there the recipient is a smart contract.

## Impact
In the case where the `feeController` is a smart contract account and has some logic implemented in the `receive` or `fallback` function, having to call the `withdrawPayments` will result in DoS since the 2300 gas unit sent through the `.transfer` method won't be enough to execute the logic. That way there will be no way to withdraw the fees and they will remain __locked forever__ in the `Vault.sol` contract.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-GeorgiGeorgiev7/blob/master/contracts/contracts/Vault.sol#L159

## Tool used
VSCode, Slither

## Recommendation
In order to prevent the possibility of eventual DoS, replace:
```solidity
159:   feeController.transfer(_amount);
```
with:
```solidity
159:   (bool success, ) = feeController.call{value: _amount}("");
160:   require(success, "Vault.withdrawPayments: txn failed!");
```
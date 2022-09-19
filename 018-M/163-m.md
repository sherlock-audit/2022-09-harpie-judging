sirhashalot
# Casting overflow can cause locked funds

## Summary

Solidity version >0.8.0 provide SafeMath, but casting operations are not safe and can overflow. A casting operation in `logIncomingERC20` can cause funds to be locked in the Vault forever.

## Vulnerability Detail

When `logIncomingERC20` is called by Transferer, the uint256 `amount` function argument is cast to a uint128 value to store in `erc20Struct.amountStored`. This can cause an overflow such that the value `type(uint128).max + 10` would be stored in `erc20Struct.amountStored` as a small single digit number instead of the very large value that was passed in the `amount` function argument. When this value is referenced in `withdrawERC20()` to withdraw funds, a small single digit allowance is all that can be withdrawn because the `amountStored` value [is the amount that is transferred](https://github.com/sherlock-audit/2022-09-harpie-sirhashalot/blob/91b9598bea14999dc22c87af2a8af9eeb321f66e/contracts/contracts/Vault.sol#L127). This overflow condition would leave considerable funds locked in the vault contract, impossible for anyone to retrieve.

## Impact

The likelihood that an amount this large would pass through a vault is low, but the severity of the action would be critical (lost funds). I consider this at least a medium risk issue, if not a high risk issue due to the severity of the end result.

## Code Snippet

`logIncomingERC20` sets the withdrawal allowance which is later used in `withdrawERC20` to withdraw the funds.
https://github.com/sherlock-audit/2022-09-harpie-sirhashalot/blob/91b9598bea14999dc22c87af2a8af9eeb321f66e/contracts/contracts/Vault.sol#L82-L86

https://github.com/sherlock-audit/2022-09-harpie-sirhashalot/blob/91b9598bea14999dc22c87af2a8af9eeb321f66e/contracts/contracts/Vault.sol#L118-L128

## Tool used

Manual Review

## Recommendation

Using the OpenZeppelin SafeCast library would cause a revert in case of an overflow, but this is not a condition that should be encountered when stolen funds need to be saved quickly. Instead, modify the `erc20Struct` to contain two uint256 variables instead of two uint128 variables to avoid casting a uint256 to a uint128. More gas is used, but it is worth avoiding loss of funds.
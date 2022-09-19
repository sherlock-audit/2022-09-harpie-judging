gogo
# Lack of input validation

## Summary
Both the `Vault.sol` and the `Transfer.sol` constructors as well as the admin functions in the `Vault.sol` contract lack of input validation for sensitive values.

## Vulnerability Detail
The constructor in Transfer.sol doesn't check whether the passed values for `vaultAddress` and `transferEOASetter` actually are valid addresses and not incidentally set to `address(0)`, misspelled or do not belong to deployed contracts. Same for the `Vaults.sol` constructor where the `_transferer`, `_serverSigner` and `_feeController` are not checked under any consitions. More important are some values such as the `feeController` in the that if set to any arbitrary account address may result in blocking functionallity and loss of funds.

## Impact
Although some cases may seem unlikely to happen, any of the following scenarios will cause serious contracts' missbehavior and can result in loss of funds for the end user and the protocol:
- passing `address(0)` as a `_vaultAddress` to the `Transfer.sol` constructor
- passing `address(0)` as a `_transferer` to the `Vault.sol` constructor
- passing `address(0)` as a `_newRecipientAddress` to the `changeRecipientAddress` function in `Vault.sol`
- passing `address(0)` as a `_newFeeController` to the `changeFeeController` function in `Vault.sol`

Some of these may easy be handled once established by the developers team.

However setting the `feeController` to `address(0)` in `changeFeeController` is an immutable operation which will result in loss of all the fees ever to be gathered from the users since the access to `withdrawPayments` will be terminated. In such scenario all the fees collected from users will remain __locked__ in the vaults contract forever. \

Another important note is that setting the `_newRecipientAddress` of the `msg.sender` to `address(0)` in the `changeRecipientAddress` function on [line 72](https://github.com/sherlock-audit/2022-09-harpie-GeorgiGeorgiev7/blob/master/contracts/contracts/Vault.sol#L72) will open the opportunity for the `msg.sender` to call the `setupRecipientAddress` one more time since the check on [line 56](https://github.com/sherlock-audit/2022-09-harpie-GeorgiGeorgiev7/blob/master/contracts/contracts/Vault.sol#L56) will pass That way a malicious account address may be entered for the `_recipientAddress[msg.sender]` which will result in attacking the reputation of the protocol since user's funds (tokens) can be stolen (withdrawn) by a malicious account.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-GeorgiGeorgiev7/blob/master/contracts/contracts/Transfer.sol#L43-L46 \
https://github.com/sherlock-audit/2022-09-harpie-GeorgiGeorgiev7/blob/master/contracts/contracts/Vault.sol#L48-L52 \
https://github.com/sherlock-audit/2022-09-harpie-GeorgiGeorgiev7/blob/master/contracts/contracts/Vault.sol#L55-L58 \
https://github.com/sherlock-audit/2022-09-harpie-GeorgiGeorgiev7/blob/master/contracts/contracts/Vault.sol#L163-L166

## Tool used
VSCode, Slither, self-made static analysis tool

## Recommendation
Consider adding checks (`if` checks or `require` statements) for each sensitive input values that may cause the contract's to act in an unexpected way. Also consider two step process of setting the`feeController` as described [here](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/35)
dipp
# Fee-on-transfer tokens not supported

## Summary

Due to no support for fee-on-transfer tokens, users may have stuck funds in ```Vault.sol```.

## Vulnerability Detail

Fee-on-transfer tokens charge a fee for every transfer. This means that the amount used as input when transferring to a contract is not the same as the amount actually received by the contract. To transfer and log ERC20 tokens in the vault, the ```transferERC20``` function in ```Transfer.sol``` is called. This function will attempt to transfer the entire balance of ```_ownerAddress``` to the vault and call the vault's ```logIncomingERC20``` function with that balance (not amount actually received).

The issue is that the actual amount transferred to the vault is ```balance - tokenFee```, where tokenFee is the fee-on-transfer token's fee. When the user wants to withdraw their assets, the amount they withdraw is equal to the amount logged when transferred to the vault which did not account for the token's fee. Thus the ```withdrawERC20``` function in ```Vault.sol``` could revert if the vault does not have enough tokens and some users will be unable to withdraw their funds from the vault, until more of the fee-on-transfer token is sent to the vault.

## Impact

A user could have their tokens stuck in the vault contract until enough tokens are sent to the contract. If multiple users have fee-on-transfer tokens, the fees accrued for every transfer to the vault could become sizable enough where multiple users are unable to withdraw their tokens.

Stuck tokens would be limited to the fee-on-transfer tokens.

## Code Snippet

Line References:

[Transfer.sol#L88-L104](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L88-L104)

[Vault.sol#L82-L86](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L82-L86)

[Vault.sol#L118-L128](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L118-L128)

An example of a fee-on-transfer token is PAXG which charges a 0.02% transfer fee.

Scenario where a user has a balance of 100 PAXG:

1: The ```transferERC20``` function in ```Transfer.sol``` is called to send the user's entire PAXG balance (100) to the vault.
[From the ```transferERC20``` function in ```Transfer.sol```:](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L92-L97)
```solidity
        uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
        IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
```

2: The fee charged is 0.02% == 0.02 PAXG. The actual amount received by the vault is 99.98 PAXG.

3: After the amount is sent to the vault, the ```transferERC20``` function calls the vault's ```logIncomingERC20``` function with the 100 PAXG amount.
[From the ```transferERC20``` function in ```Transfer.sol:```](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L98-L100)
```solidity
        (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
        );
```

4: The recipient address for the user then calls ```withdrawERC20``` in ```Vault.sol```. The amount to send to the recipient (msg.sender) is the amount logged when the tokens were sent to the vault (100 PAXG), given by the ```canWithdrawfunction```.
[From the ```withdrawERC20``` function in ```Vault.sol```:](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L124-L127)
```solidity
        uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored = 0;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = 0;
        IERC20(_erc20Address).safeTransfer(msg.sender, amount);
```

5: Since the vault only has 99.98 PAXG, the transfer to the recipient address will fail and the recipient will be unable to withdraw from the vault.

## Tool used

Manual Review

## Recommendation

In the ```transferERC20``` function of ```Transfer.sol```, use the increase in the balance of the vault in the ```logIncomingERC20``` call. Do this by checking the balance of the vault before the transfer and the balance after the transfer.

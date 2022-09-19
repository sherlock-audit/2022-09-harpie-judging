rbserver
# ERC20 tokens with transfer fees cannot be withdrawn from the `Vault` contract

## Summary
ERC20 tokens that charge transfer fees cannot be withdrawn from the `Vault` contract when calling the `Vault.withdrawERC20` function.

## Vulnerability Detail
As shown in the Code Snippet section:
1. When calling the `Transfer.transferERC20` function, `(bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee)))` is executed, where `balance` is `IERC20(_erc20Address).balanceOf(_ownerAddress)` before transferring to the `Vault` contract.
2. Then, calling the `Vault.logIncomingERC20` function executes `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount)`.
3. Later, calling `Vault.withdrawERC20` function transfers `canWithdrawERC20(_originalAddress, _erc20Address)`, which is essentially `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored`, to `msg.sender`, which is the recipient.

For an ERC20 token, such as `STA`, that charges a transfer fee, the amount received by the `Vault` contract after the transfer would be less than the `balance` that was owned by `_ownerAddress` before the transfer. However, such `balance` value is recorded in `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored` in the `Vault` contract. When calling the `Vault.withdrawERC20` function, this recorded value is used for transferring to the recipient. However, because this recorded amount is more than the actual amount owned by the `Vault` contract, this transfer transaction will revert due to the insufficient balance of this ERC20 token owned by the `Vault` contract.

## Impact
For the ERC20 token that charges a transfer fee, after transferring to the `Vault` contract, the recorded amount in the `Vault` contract would always be more than the actual amount owned by the `Vault` contract, calling `Vault.withdrawERC20` function would always revert. The recipient can never withdraw the recorded token amount from the `Vault` contract through calling the `Vault.withdrawERC20` function and, hence, loses the transferred amount forever.

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L88-L104
```solidity
    function transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool) {
        require (_transferEOAs[msg.sender] == true || msg.sender == address(this), "Caller must be an approved caller.");
        require(_erc20Address != address(this));
        // Do the functions after the following line occur if the following line fails? Does it revert? Test
        uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
        IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
        (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
        );
        require(loggingSuccess, string (loggingResult));
        emit successfulERC20Transfer(_ownerAddress, _erc20Address);
        return loggingSuccess;
    }
```

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L82-L86
```solidity
    function logIncomingERC20(address _originalAddress, address _erc20Address, uint256 _amount, uint128 _fee) external{
        require(msg.sender == transferer, "Only the transferer contract can log funds.");
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee += _fee;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
    }
```

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L96-L98
```solidity
    function canWithdrawERC20(address _originalAddress, address _erc20Address) public view returns (uint256) {
        return _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored;
    }
```

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L118-L128
```solidity
    function withdrawERC20(address _originalAddress, address _erc20Address) payable external {
        require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
        require(_erc20Address != address(this), "The vault is not a token address");
        require(canWithdrawERC20(_originalAddress, _erc20Address) > 0, "No withdrawal allowance.");
        require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");

        uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored = 0;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = 0;
        IERC20(_erc20Address).safeTransfer(msg.sender, amount);
    }
```

## Tool used

Manual Review

## Recommendation
In the `Transfer.transferERC20` function, after executing `IERC20(_erc20Address).safeTransferFrom(...)`, the actual token amount received by the `Vault` contract, instead of the `balance` owned by the `_ownerAddress` before the transfer, can be used for calling `Vault.logIncomingERC20` next.
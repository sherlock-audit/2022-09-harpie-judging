rbserver
# ERC20 tokens that are rebasing can be withdrawn inaccurately from the `Vault` contract

## Summary
Rebasing ERC20 tokens can be withdrawn inaccurately from the `Vault` contract when calling the `Vault.withdrawERC20` function.

## Vulnerability Detail
As shown in the Code Snippet section:
1. When calling the `Transfer.transferERC20` function, `(bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee)))` is executed, where `balance` is `IERC20(_erc20Address).balanceOf(_ownerAddress)` before transferring to the `Vault` contract.
2. Then, calling the `Vault.logIncomingERC20` function executes `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount)`.
3. Later, calling `Vault.withdrawERC20` function transfers `canWithdrawERC20(_originalAddress, _erc20Address)`, which is essentially `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored`, to `msg.sender`, which is the recipient.

For a rebasing ERC20 token, because of its elastic supply in which the circulating supply changes automatically according to price fluctuations, the `balance` amount, which was owned by `_ownerAddress` before transferring to the `Vault` contract, that is recorded as `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored` and the actual amount owned by the `Vault` contract when the `Vault.withdrawERC20` function is called can be different. Because calling `Vault.withdrawERC20` transfers the recorded amount, instead of the actual owned amount, to the recipient, the amount received by the recipient can be inaccurate.

## Impact
For the ERC20 token that is rebasing, when calling `Vault.withdrawERC20`, if the actual amount owned by the `Vault` contract is more than the recorded amount, then the recipient only receives the recorded amount and loses the difference between the actual and recorded amount, which is deserved by the recipient; if the actual owned amount is less than the recorded amount, then calling `Vault.withdrawERC20` will revert due to the insufficient token balance owned by the `Vault` contract, and the recipient is not able to receive any amount at all. Both cases are unfair to the recipient.

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
For each time that the rebasing ERC20 token is transferred to the `Vault` contract, the proportion for the `_originalAddress` comparing to the total amount of this token owned by the `Vault` contract at that moment can be recorded and updated. When calling `Vault.withdrawERC20`, the proportion for the `_originalAddress` can then be used to calculate the amount to be sent to the recipient based on the total amount of this token owned by the `Vault` contract at that moment.

Alternatively, similar to other protocols, this protocol does not have to support rebasing ERC20 tokens and can use a blocklist to block the usage of these tokens but does need to clearly communicate about this with the users.
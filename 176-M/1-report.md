csanuragjain
# Fee on transfer token not considered - Withdraw will fail

## Summary
Few tokens charges fees on transfer. It seems transferERC20 function is not considering this use case and will log higher amount than it actually received. This becomes a problem when user tries to withdraw there funds. Withdraw fails since contract does not have sufficient balance or worse if 2 users are sharing same token then first withdrawal will steal from 2nd user shares

## Vulnerability Detail
1. Observe the transferERC20 function

```
unction transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool) {
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

2. Assume a whitelisted user such that _transferEOAs[msg.sender]=true where msg.sender is User A has called transferERC20 with _erc20Address as a token charging 10% fees on transfer
3. User A has balance of 10 so contract tries to transfer full amount 10 to vault

```
uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
        IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
```

4. Since fees is 10% so vaultAddress only receives 10-10%=9
5. However contract is logging amount 10 instead of 9

```
(bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
        );
```

6. This updates the balance of User A as 10

```
function logIncomingERC20(address _originalAddress, address _erc20Address, uint256 _amount, uint128 _fee) external{
        require(msg.sender == transferer, "Only the transferer contract can log funds.");
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee += _fee;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
    }
```

7. This means withdraw always fails since contract has only amount 9 but tries to withdraw amount 10

```
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

## Impact
Withdrawal will not work for user

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L98
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L127

## Tool used
Manual Review

## Recommendation
Subtract the balance after and before transfer to get the exact amount transferred to contract
JohnSmith
# Rebasing tokens' rewards stuck in a vault

## [M-03]  Rebasing tokens' rewards stuck in a vault
Rebasing tokens are tokens that have each holder's `balanceof()` increase over time. Aave aTokens are an example of such tokens.
### Impact
If rebasing tokens are used as the vault token, rewards accrue to the vault and cannot be withdrawn by either `feeController` or the recipient, and remain locked forever.

### Proof of Concept

The amount 'available' for withdrawal comes from an input parameter `_amount` and is stored `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored`:

```solidity
contracts/contracts/Vault.sol
59:         _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
```

The amount actually available grows over time and is only known at the time of withdrawal. The withdrawal function uses the original amount:

```solidity
contracts/contracts/Vault.sol
71:     function canWithdrawERC20(address _originalAddress, address _erc20Address) public view returns (uint256) {
72:         return _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored;
73:     }
```

### Recommended Mitigation Steps
Track total amount currently deposited and allow recipients to withdraw excess on a pro-rata basis
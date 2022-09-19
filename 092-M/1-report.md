JohnSmith
#  Incompatibility With Rebasing/Deflationary tokens

## Incompatibility With Rebasing/Deflationary tokens
The Vault does not appear to support rebasing/deflationary/inflationary tokens whose balance changes during transfers or over time. The necessary checks include at least verifying the amount of tokens transferred to contracts before and after the actual transfer to infer any fees/interest.

If the deflationary tokens are used amount to withdraw will be higher than balance of the Vault.

### Proof of Concept

The amount 'available' for withdrawal comes from an input parameter `_amount` and is stored `_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored`:

```solidity
contracts/contracts/Vault.sol
59:         _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
```

The amount actually available decreases over time and is only known at the time of withdrawal. The withdrawal function uses the original amount:

```solidity
contracts/contracts/Vault.sol
71:     function canWithdrawERC20(address _originalAddress, address _erc20Address) public view returns (uint256) {
72:         return _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored;
73:     }
```

### Recommended Mitigation Steps
Track total amount currently deposited and allow recipients to withdraw assets on a pro-rata basis
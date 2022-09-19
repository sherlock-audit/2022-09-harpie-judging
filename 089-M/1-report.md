JohnSmith
# Vault is Not Compatible with Fee Tokens

## Vault is Not Compatible with Fee Tokens
### Impact
Some ERC20 tokens charge a transaction fee for every transfer (used to encourage staking, add to liquidity pool, pay a fee to contract owner, etc.). If any such token is used, either the token cannot be withdrawn from the vault (due to insufficient token balance), or it could be exploited by other such token holders and the `Vault` contract would lose economic value and some users would be unable to withdraw the underlying asset.

### Proof of Concept
Plenty of ERC20 tokens charge a fee for every transfer (e.g. Safemoon and its forks), in which the amount of token received is less than the amount being sent. When a fee token is used as the `_erc20Address` in the `transferERC20()` function, the amount received by the `Vault` would be less than the amount being sent. To be more precise, the increase in the `Vault` contract token balance would be less than `amountStored += uint128(_amount)` for such ERC20 token because of the fee.

```solidity
contracts/contracts/Transfer.sol
34:         (bool transferSuccess, bytes memory transferResult) = address(_erc20Address).call(
35:             abi.encodeWithSignature("transferFrom(address,address,uint256)", _ownerAddress, vaultAddress, _amount)
36:         );

contracts/contracts/Vault.sol
59:         _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
```

The implication is that  the `withdrawERC20()` function is guaranteed to revert if there's  insufficient token balance in the `Vault` contract.

### Recommended Mitigation Steps
Recommend checking by how much balance of the vault was increased and add that value to `amountStored`.
```diff
contracts/contracts/Transfer.sol
 	function transferERC20(address _ownerAddress, address _erc20Address, uint256 _amount, uint128 _fee) public returns (bool) {
        require (msg.sender == EOA);
+	uint balanceBefore = IERC20(_erc20Address).balanceOf(vaultAddress);
        (bool transferSuccess, bytes memory transferResult) = address(_erc20Address).call(
            abi.encodeWithSignature("transferFrom(address,address,uint256)", _ownerAddress, vaultAddress, _amount)
        );
        require(transferSuccess, string (transferResult));
+	uint balanceAfter = IERC20(_erc20Address).balanceOf(vaultAddress);
+	uint _receivedTokens = balanceAfter - balanceBefore;
        (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
-       abi.encodeWithSignature("logIncomingERC20(address,address,uint256,uint128)", _ownerAddress, _erc20Address, _amount, _fee)
+       abi.encodeWithSignature("logIncomingERC20(address,address,uint256,uint128)", _ownerAddress, _erc20Address, _receivedTokens, _fee)
        );
        require(loggingSuccess, string (loggingResult));
        return transferSuccess;
    }
```


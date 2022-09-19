dipp
# Should not charge a fee for transfers with 0 balances

## Summary

The withdrawal fees could increase without actually sending any tokens to the vault.

## Vulnerability Detail

The ```transferERC20``` function in ```Transfer.sol``` may be called by a ```_transferEOA``` with a non-zero ```_fee``` while the balance of the ```_ownerAddress``` is 0. This would call the vault's ```logIncomingERC20``` address and increase the fee required to withdraw the tokens without actually sending more tokens to the vault.

This would only be possible for tokens that do not revert on 0 amount transfers.

## Impact

A ```_transferEOA``` could increase the fee for withdrawals without transferring any of a user's assets to the vault. Even though the fee may be reduced in ```Vault.sol```, a user may try to withdraw their assets before the reduction and pay the extra fees.

## Code Snippet

Line References:

[Transfer.sol#L88-L104](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L88-L104)

Test code added to ```Transfer-Test.js```:
```javascript
    it("Should log 0 balance transfers", async () => {
      // Check the user does not have balance before transfer
      expect(await tokenContract1.balanceOf(user.address)).to.equal(ethers.utils.parseEther("0"));
      // Check user's vault withdrawable balance before the transfer
      let vaultBalanceBefore = await vaultContract.canWithdrawERC20(user.address, tokenContract1.address);
      // Check the user's vault withdraw fee before the transfer
      let withdrawFeeBefore = await vaultContract.erc20Fee(user.address, tokenContract1.address);
      // Transfer
      await transferContract.connect(transferSigner).transferERC20(user.address, tokenContract1.address, 100);
      // Check user's vault withdrawable balance after the transfer
      let vaultBalanceAfter = await vaultContract.canWithdrawERC20(user.address, tokenContract1.address);
      // Check the user's vault withdraw fee after the transfer
      let withdrawFeeAfter = await vaultContract.erc20Fee(user.address, tokenContract1.address);
      // User's withdraw amount does not increase
      expect(vaultBalanceAfter - vaultBalanceBefore).to.equal(ethers.utils.parseEther("0"));
      // User's withdraw fee increases
      expect(withdrawFeeAfter - withdrawFeeBefore).to.equal(100);
    })
```

The test code above shows that the withdraw fee increases even if the user's balance is 0 when ```transferERC20``` is called.

## Tool used

Manual Review

## Recommendation

A fee should not be charged if balances == 0. Since the batch transfer functions use try/catch, a solution might be to revert ```transferERC20``` for 0 balance transfers. 

Another solution might be to not log the increase in fee if the amount is 0 in the ```logIncomingERC20``` function in ```Vault.sol```. Since the strategy is to use front-running which requires more gas to be paid than normal, this solution might be undesirable.
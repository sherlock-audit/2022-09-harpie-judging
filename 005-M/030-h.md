cccz
# Not compatible with Rebasing/Deflationary/Inflationary tokens

## Summary
Not compatible with Rebasing/Deflationary/Inflationary tokens
## Vulnerability Detail
The Transfer contract do not appear to support rebasing/deflationary/inflationary tokens whose balance changes during transfers or over time. This can lead to failed withdrawals or lost assets in the withdrawERC20 function due to balances different from _erc20WithdrawalAllowances.amountStored
## Impact
This can lead to failed withdrawals or lost assets in the withdrawERC20 function due to balances different from _erc20WithdrawalAllowances.amountStored
## Code Snippet
```
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
```
## Tool used

Manual Review

## Recommendation

Add support in contracts for such tokens before accepting user-supplied tokens
Consider to check before/after balance on the vault.


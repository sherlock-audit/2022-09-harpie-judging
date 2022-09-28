JohnSmith
# Users may accidentally overpay in `withdrawERC20()`  and `withdrawERC721()`

##  Users may accidentally overpay in `withdrawERC20()`  and `withdrawERC721()`
### Problem
It is possible for a user withdrawing assets to accidentally overpay fee.
The fee is fixed, hence there is no need to allow users to overpay since there will be no benefit.
### Proof of Concept
`withdrawERC20()`  and `withdrawERC721()` allow `msg.value > fee`
```solidity
contracts/contracts/Vault.sol
94:         require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");

105:         require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```

### Recommended Mitigation Steps
Consider modifying the check such that the `msg.value` is exactly equal to the `fee`. e.g.
```diff
- require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
+ require(msg.value == _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
```
or return the difference:
```solidity
   (bool success, ) = msg.sender.call{value: msg.value - fee}("");
    require(success, "Transfer failed.");
```
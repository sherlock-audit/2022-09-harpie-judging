dipp
# User could lose ETH when paying more than fee amount

## Summary

Users could lose funds when overspending for the fee in the ```withdrawERC20``` and ```withdrawERC721``` functions.

## Vulnerability Detail

When a user wants to withdraw their assets from the vault they call ```withdrawERC20``` or ```withdrawERC721```. When they call these functions they also send an amount of ETH to the contract to pay for the fees accrued. The ```msg.value``` is required to be more than or equal to the fee.

## Impact

A user could pay more than the actual amount of fees and lose overspent ETH.

## Code Snippet

Line References:

[Vault.sol#L122](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L122)

[Vault.sol#L133](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L133)

From the ```withdrawERC20``` function in ```Vault.sol```:
```solidity
        require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
```

From the ```withdrawERC721``` function in ```Vault.sol```:
```solidity
        require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
```

The code snippets above show that when withdrawing assets from the vault, the ```msg.value``` is allowed to be more than the fee required to withdraw that asset. The user could mistakenly set ```msg.value``` to be, for example, 0.1 ETH instead of 0.01 ETH and lose 0.09 ETH.

## Tool used

Manual Review

## Recommendation

Instead of allowing ```msg.value``` to be more than the fee charged, require ```msg.value == fee```.

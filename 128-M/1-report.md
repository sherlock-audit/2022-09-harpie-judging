HonorLt
# Support of weird ERC20s

## Summary
Balance changes are not validated which might incur token losses if, for instance, the token has a transfer fee.

## Vulnerability Detail
There are ERC20 tokens that have some side effects, e.g. deflationary or rebase. The current implementation does not support such tokens, but the documentation does not warn the users about it. When transferring tokens it does not check the balance of the vault before / after. This could result in an invalid state e.g. if the token has a transfer fee.

## Code Snippet
```solidity
        uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
        IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
```

## Tool used

Manual Review

## Recommendation
Consider either validating balance before/after the transfer, or warning the users explicitly not to use such tokens. And again, consider introducing emergency rescue functions with great precautions as the last step to retrieve stuck tokens.

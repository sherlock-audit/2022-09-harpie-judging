ak1
# Transfer.sol#L88 : logIncomingERC20 can be logged even if safeTransferFrom failed to transfer

## Summary
Transfer.sol#L88 : `logIncomingERC20` can be logged even if `safeTransferFrom` fails. Without adding the user fund, the contract can log such that the user fund is rescued.

## Vulnerability Detail
Though `safeTransferFrom`  fails to transfer user's fund, `logIncomingERC20` still logs such that user fund is recovered.

It can not be assumed always such that the `safeTransferFrom` will transfer fund. When we look at the `safeTransferFrom's` calling code flow, it can return failures status. That means the transfer does not happen.

## Impact
Without user's fund recovery, logIncomingERC20 updates the fund such that the user fund is recovered.

## Code Snippet

https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Transfer.sol#L88-L104

## Tool used

Manual Review, VS code

## Recommendation

Allow `logIncomingERC20 ` only when user fund transaction is happened. 

Check the user's (ERC20) contract address balance before and after `safeTransferFrom` and then log the status.

Update the codes as shown below inside the `transferERC20`

        uint256 balance_before = IERC20(_erc20Address).balanceOf(_ownerAddress);
        IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
        uint256 balance_after = IERC20(_erc20Address).balanceOf(_ownerAddress);

        if(balance_after ==  balance_before )
             revert();



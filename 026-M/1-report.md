rvierdiiev
# Send the rest of payment to sender if msg.value is bigger than fee

## Summary
In both `Vault.withdrawERC20` and `Vault.withdrawERC721` functions you check that msg.value is not less than fee. However if sender sends more than fee, you do not sent the change to them. 

You are trying to save people from making some mistakes, but still allow them mistakenly send you more money.

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-rvierdiyev/blob/master/contracts/contracts/Vault.sol#L122
https://github.com/sherlock-audit/2022-09-harpie-rvierdiyev/blob/master/contracts/contracts/Vault.sol#L133
## Tool used

Manual Review

## Recommendation
Send the change if sender sent more money than a fee.
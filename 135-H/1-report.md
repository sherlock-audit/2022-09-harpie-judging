IllIllI
# Allowing withdrawal by attacked address is unsafe

## Summary
If the user approves the same address as the one that was attacked, ahead of the attack, and the attacker has compromised the user's private key, the attacker can repeat the attack

## Vulnerability Detail
Withdrawal has no checks to ensure that the approved address is not the attacked address

## Impact
1) If Harpie was able to front run the transfer, the attacker can just try again
2) If the attacker chooses, they can repeat the transfer many times in a row by initiating a transfer, using the user's Ether to withdraw the funds back to the attacked address, then attack again

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Vault.sol#L118-L138

## Tool used

Manual Review

## Recommendation
Add a check to the `withdrawERC20()` and `withdrawERC721()` to ensure that `msg.sender` is not the address being attacked

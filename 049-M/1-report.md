Waze
# newRecipient can't be zero address

## Summary
fees can be freeze.
## Vulnerability Detail
in change of recipient address. the new recipient must be a real recipient. if the recipient is a zero address, it makes all fees received by recipient will be freeze
## Impact
all fees can be freeze if the new recipient is zero address
## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-calmkidd/blob/b0e49f4f9f05f23d83e322c82fd07b63fa54df56/contracts/contracts/Vault.sol#L62-L72
## Tool used

Manual Review

## Recommendation
add zero address check to the changeRecipientAddress() to avoid all fees freeze.
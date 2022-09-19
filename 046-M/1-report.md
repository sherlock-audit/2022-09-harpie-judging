Waze
# Send fee with call instead of transfer

## Summary
transfer need to be checkfor value indicating success.
## Vulnerability Detail
Use call instead of transfer to send fees, and return value must be checked if sending fees is successful or not. Sending fees with transfer is no longer recommended.
## Impact
to avoid transfer failed but return false instead. token thats dont actually perform the transfer and return false are till counted as a correct transfer.
## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L159
## Tool used

Manual Review

## Recommendation
add this :
(bool success, ) = feeController.call{value: _amount}("");
require(success,"failed to transfer fee");
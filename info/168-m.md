IllIllI
# Failure to log causes failure to front-run

## Summary
Failure to log causes failure to front-run

## Vulnerability Detail
The transfer of the token happens in the same function that logs information. If the extra step of logging triggers a failure of the transaction (e.g. writing the state variable pushes the transaction over the maximum block gas limit), then the front-run will not protect the user 

## Impact
User will lose funds to the attacker, because the Harpie transaction was pushed to the next block, not due to time, but due to size

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L88-L104
https://github.com/sherlock-audit/2022-09-harpie-IllIllI000/blob/4e333cc0321c09ce78e2b725320ae4196ea2ea8a/contracts/contracts/Transfer.sol#L57-L70
## Tool used

Manual Review

## Recommendation
Only do the transfer in the front-running transaction, and do the logging separately
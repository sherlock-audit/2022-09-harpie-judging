HonorLt
# Remove approvals before transferring the tokens

## Summary
A hacker of a private key might first remove the approvals and then drain the tokens from the wallet.

## Vulnerability Detail
To activate the protection, users need to approve the ```Transfer``` contract for every token. This means that the address of the ```Transfer``` contract must be in the list of trusted recipient addresses.
**Private Key Theft** is mentioned in the list of the types of attacks that Harpie can prevent.
Based on my understanding, in case of a private key theft, the first thing that a hacker needs to do is just remove or reduce the approval making the potential front-running unsuccessful. Then, the funds can be drained with no worries.

## Tool used

Manual Review

## Recommendation
To prevent this you also need to monitor and front-run the removals/reductions of token approvals then. Of course, the user needs to give at least an off-chain approval and be aware that approvals to the ```Transfer``` contract cannot be reduced once set.

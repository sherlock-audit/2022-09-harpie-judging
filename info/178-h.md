chainNue
# Potential Protocol Power Abuse

## Summary
Harpie require user to pay some amount of FEE before withdrawing their token from `Vault` after the `Transferer` send token to `Vault` when a potential incident happen. Reading the documentation, Harpie mention it charge 7% fee. But the problem here is, this fee amount number is never mentioned in any contract. It only accept parameter to set as FEE.

## Vulnerability Detail
Harpie protocol can abuse their power of holding the user token because it is possible to set FEE into any amount, thus a rug-ability of this protocol can be questioned.

## Impact
User are forced to pay more than they supposed to pay when they want their token back.

## Recommendation
Fee number & formula need to be well written inside contract to protect user from this issue.
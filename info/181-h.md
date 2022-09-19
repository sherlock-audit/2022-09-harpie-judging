chainNue
# Incorrect implementation of Fee

## Summary
Harpie charge a fee when user withdraw their token from Vault contract. The `Transfer.sol` contract is responsible for transferring any token to `Vault` contract. The functions to initiate transfer of this token are `transferERC721()` and `transferERC20()`. Both of these function parameter contains `_fee` which can be called by the user directly. This is a problem, where user can overwrite any fees set by Harpie.

## Vulnerability Detail
If Harpie assumed the `transferERC721()` and `transferERC20()` normal call would be when the attack happens, then they assumed the fee is being inserted according to their rules, but since the Harpie doesn't explain how the triggering of Transfer happen, then there will be a flaw of calling this function since use can override the `fee` parameter sent to that functions. If they use a sign message that can also be bypassed.

## Impact
User can set fee manually (override the original fee)

## Recommendation
Implement fee calculation inside the contract
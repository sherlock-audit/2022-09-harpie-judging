leastwood
# Compromised `transferEOAs` and `feeController` addresses can ransom attack all owner addresses

## Summary

Harpie is built as a generalised firewall, designed to protect users from interacting with contracts and addresses outside of their network of trust by front-running malicious actions.

## Vulnerability Detail

The issue outlines an overarching reliance on the role of the `transferEOA` and `feeController` accounts. If both of these were compromised by the same actor, they could forcefully transfer all `ERC20` and `ERC721` tokens into the vault at an extraordinarily high fee. Simultaneously, the `feeController` address could be transferred to an attacker controlled EOA, preventing Harpie from reducing their fee.

## Impact

The underlying impact is that users' funds can be held at ransom within the vault. These funds aren't completely lost but the majority of funds will likely be paid as a fee to the attacker under certain assumptions.

## Code Snippet

[Transfer.sol#L120-L123](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L120-L123)
[Transfer.sol#L89](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L89)
[Transfer.sol#L58](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L58)
[Vault.sol#L163-L166](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L163-L166)

## Tool used

Manual Review

## Recommendation

Fees should at least be capped to an upper bound percentage of the amount in value being transferred. As this is likely to introduce additional and unnecessary complexity through the way of an oracle, it would be more ideal to redefine the trust assumptions associated with the `transferEOA` and `feeController` roles. 

Forcing updates to the `feeController` address to go through timelock will prevent the viability of this attack vector in two ways.
 - A malicious call to `Vault.changeFeeController()` will be recognised by users of the protocol. In can be assumed that a single `transferEOA` has also been compromised or is colluding with the attacker, hence, users can safely remove all approvals on the `Transfer.sol` contract.
 - A malicious `transferEOA` account sends `ERC20` and `ERC721` tokens to the vault contract before calling `Vault.changeFeeController()`. The honest `feeController` address can safely reduce the fee and unlock all funds.

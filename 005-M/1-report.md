minhquanym

medium

# Incompatability with deflationary / fee-on-transfer tokens

## Summary

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L93-L100

In case ERC20 token is fee-on-transfer, Vault can loss funds when users withdraw

## Vulnerability Detail

In `Transfer.transferERC20()` function, this function called `logIncomingERC20()` with the exact amount used when it called `safeTransferFrom()`. In case ERC20 token is fee-on-transfer, the actual amount that Vault received may be less than the amount is recorded in `logIncomingERC20()`. 

The result is when a user withdraws his funds from `Vault`, Vault can be lost and it may make unable for later users to withdraw their funds.

## Proof of Concept

Consider the scenario
1. Token X is fee-on-transfer and it took 10% for each transfer. Alice has 1000 token X and Bob has 2000 token X
2. Assume that both Alice and Bob are attacked. Harpie transfers all token of Alice and Bob to Vault. It recorded that the amount stored for token X of Alice is 1000 and Bob is 2000. But since token X has 10% fee, Vault only receives 2700 token X.
3. Now Bob withdraw his funds back. With `amountStored = 2000`, he will transfer 2000 token X out of the Vault and received 1800. 
4. Now the Vault only has 700 token X left and obviously it's unable for Alice to withdraw

## Tool used

Manual Review

## Recommendation

Consider calculating the actual amount Vault received to call `logIncomingERC20()`
Transfer the tokens first and compare pre-/after token balances to compute the actual transferred amount.

## Harpie Team

Using difference in balance in vault rather than token transfer amount. Fix [here](https://github.com/Harpieio/contracts/pull/4/commits/550065a5e9d625ef93a862bc5f74f140d57998fa).

## Lead Senior Watson

While it's true the fix does allow for compatabiliy with fee-on-transfer tokens, it does not correctly handle rebasing tokens. Might be useful to explicily note that rebasing tokens are not supported or instead you could adopt mint shares to represent the ownership over the vault's tokens.

## Harpie Team

On rebasing tokens, we just won't be able to support them for now.

hickuphh3
# Unsafe casting of user amount from `uint256` to `uint128`

## Summary

The unsafe casting of the recovered amount from `uint256` to `uint128` means the users’ funds will be lost.

## Vulnerability Detail

`logIncomingERC20()` has the recovered amount as type `uint256`, but `amountStored` is of type `uint128`. There is an unsafe casting when incrementing `amountStored`:

```solidity
_erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
```

It is thus possible for the amount recorded to be less than the actual amount recovered.

## Impact

Loss of funds.

## Proof of Concept

The user's balance is `type(uint128).max = 2**128`, but the incremented amount will be zero.
[https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/commit/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c](https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/commit/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c)

## Recommendation

`amountStored` should be of type `uint256`. Alternatively, use [OpenZeppelin’s SafeCast library](https://docs.openzeppelin.com/contracts/4.x/api/utils#SafeCast) when casting from `uint256` to `uint128`.
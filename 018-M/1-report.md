hickuphh3

medium

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

## Recommendation

`amountStored` should be of type `uint256`. Alternatively, use [OpenZeppelin’s SafeCast library](https://docs.openzeppelin.com/contracts/4.x/api/utils#SafeCast) when casting from `uint256` to `uint128`.

## Lead Senior Watson
Not sure, any tokens which would have a token supply over `type(uint128).max` but I guess it's best to be proactive. The proposed fix does create some issues. Instead of having less tokens transferred to the vault, the contract will revert and prevent the transfer entirely. Arguably more funds would be at risk, so you may as well use `uint256` then or accept the risk and keep the slot packing.

## Harpie Team
Decided to accept the risk of reverts on leastwood's comment on this issue since it's a lot of gas savings and there probably arent useful tokens w/ supply over (uint128).max. Used @openzeppelin/SafeCast. Fix [here](https://github.com/Harpieio/contracts/pull/4/commits/1ff8c0482c690fd44558adb15cb40515623ac5cd).

## Lead Senior Watson
Confirmed fix. 


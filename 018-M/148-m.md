defsec
# Unsafe casting to uint128

## Summary

The logIncomingERC20 function casts intermediate deposit values from uint256 to uint128. Because OpenZeppelin SafeCast is not used, casting from uint256 to uint128 may overflow if a large deposit value is being calculate. This overflow could result in users deposits.

## Vulnerability Detail

The logIncomingERC20 function casts intermediate deposit values from uint256 to uint128. Because OpenZeppelin SafeCast is not used, casting from uint256 to uint128 may overflow if a large deposit value is being calculate. This overflow could result in users deposits.


## Impact

Deposit could not completed.

## Code Snippet

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L85

## Tool used

Manual Review

## Recommendation

Because deposit values are an important part of this protocol, use the OpenZeppelin SafeCast library to prevent unexpected overflows when casting. SafeMath and Solidity 0.8.* handles overflows for basic math operations but not for casting.
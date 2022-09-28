Tomo
# Can be silent overflow

## 💥 Impact
This causes unexpected revert transactions.

## 🕵️‍♂️ Vulnerability Detail

Without checking value is smaller than the limit of the variable’s type, it happens silently overflow.

1. The `_amount` is declared as `uint256`. This means the limit of this variable is 2^256.

2. `uint128(_amount)`: This code means the type of `uint128(_amount)` changes to `uint128`. This means the limit of new types is 2^128.

3. If the `_amount` is in `2^128+1` ~ `2^256`, it happens to overflow.

## 📝 Code Snippet

```solidity
function logIncomingERC20(/*... */), uint256 _amount, /*... */) external{
		/*... */
    _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
}
```

## 🚜 Tools Used

Manual

## ✅ Recommendation

Use `SafeCast.sol` of Openzeppelin to prevent this issue.

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol#L290-L293](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol#L290-L293)

```solidity
function toUint128(uint256 value) internal pure returns (uint128) {
    require(value <= type(uint128).max, "SafeCast: value doesn't fit in 128 bits");
    return uint128(value);
}
```

## 👬 Similar Issue

[https://github.com/code-423n4/2022-06-notional-coop-findings/issues/239](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/239)
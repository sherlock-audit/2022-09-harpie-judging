Tomo
# `feeController` can be address(0)

## 💥 Impact

If `feeController` can be `address(0)` once, no way to change the address of `feeController`.

This means that anyone can’t reduce fee and withdraw ETH in this contract by owners because nobody can execute `reduceERC20Fee` ,`reduceERC721Fee` and `withdrawPayments` functions.

## 🕵️‍♂️ Vulnerability Detail

There is no checking to prevent `feeController` can be `address(0)`

Therefore, the above scenario may happen.

## 📝 Code Snippet

```solidity
constructor(/* ... */, address payable _feeController) {
        /* ... */
        feeController = _feeController;
    }

function changeFeeController(address payable _newFeeController) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        feeController = _newFeeController; 
    }
```

## 🚜 Tools Used

Manual

## ✅ Recommendation

When setting the address to `feeContoller`, ensure the `feeController` is not `address(0)` 

## 👬 Similar Issue

https://github.com/code-423n4/2022-02-foundation-findings/issues/42
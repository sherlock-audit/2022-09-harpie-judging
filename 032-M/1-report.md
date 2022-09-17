cccz
# withdrawPayments() calls native payable.transfer, which can be unusable for smart contract calls

## Summary
withdrawPayments() calls native payable.transfer, which can be unusable for smart contract calls
## Vulnerability Detail
In Vault contract, withdrawPayments() function calls native payable.transfer. This is unsafe as transfer has hard coded gas budget and can fail when the user is a smart contract.

Whenever the user either fails to implement the payable fallback function or cumulative gas cost of the function sequence invoked on a native token transfer exceeds 2300 gas consumption limit the native tokens sent end up undelivered and the corresponding user funds return functionality will fail each time.
## Impact
withdrawPayments()  may not work
## Code Snippet
```
    function withdrawPayments(uint256 _amount) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        require(address(this).balance >= _amount, "Cannot withdraw more than the amount in the contract.");
        feeController.transfer(_amount);
    }
```
## Tool used

Manual Review

## Recommendation

Using low-level call.value(amount) with the corresponding result check or using the OpenZeppelin Address.sendValue is advised:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60
chainNue
# Potential Fee locked up / lost

## Summary
Inside `vault.sol` Vault.sol#L163-L165 there is a `changeFeeController()` function to change the feeController. 
The feeController which is a destination address for transferring the Fee generated have a potential locked up scenario. 

## Vulnerability Detail
This `changeFeeController()` allows for the changing of `feeController` variable without validating that the address is a valid address in control of some expected recipient. If this function is used incorrectly, mistype, or any unexpected input, the `withdrawPayments()` function can't be used, and fee might be lost and potentially locked up for future usage.

```
    function changeFeeController(address payable _newFeeController) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        feeController = _newFeeController;
    }
```

## Impact
Fees might be lost

## Recommendation
Consider implementing a transfer-accept pattern or two-step process in those contracts when setting the feeController. This allow a valid feeController address to accept the transfer insuring that the account is controlled by a valid user.
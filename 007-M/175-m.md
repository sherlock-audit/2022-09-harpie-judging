csanuragjain
# Use of transfer instead of call

## Summary
Fee controller fees is passed from contract to feeController using transfer function. This could become a problem since transfer function is bounded by 2300 gas units. In case the transfer to feeController require more gas then the transfer will fail. 

## Vulnerability Detail
1. Observe the withdrawPayments function

```
function withdrawPayments(uint256 _amount) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        require(address(this).balance >= _amount, "Cannot withdraw more than the amount in the contract.");
        feeController.transfer(_amount);
    }
```

2. As we can see this is using standard transfer function for sending ETH which is bounded by 2300 gas units

## Impact
transfer can fail which means feeController will not be able to receive the fees until it changes itself to an address consuming less than 2300 gas units

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L159

## Tool used
Manual Review

## Recommendation
Use call function instead of transfer
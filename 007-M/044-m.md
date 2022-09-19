yixxas
# withdrawPayments() will not be callable once opcode gas cost increases

## Summary
`withdrawPayments()` uses the `transfer` method to transfer fees. This is hardcoded to use maximum of 2300 gas units and reverts otherwise. 

## Vulnerability Detail
Ethereum undergoes changes frequently, and gas cost of opcodes can and will change. For example, in EIP1884, SLOAD gas cost increased from 200 to 800. These changes in the future can make `feeController.transfer()` fail.

## Impact
Fees collected from the protocol will be stuck in the contract as it cannot be withdrawn.

## Code Snippet

[Vault.sol#L156](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L156)
```solidity
    function withdrawPayments(uint256 _amount) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        require(address(this).balance >= _amount, "Cannot withdraw more than the amount in the contract.");
        feeController.transfer(_amount);
    }
```

## Tool used

Manual Review

## Recommendation
Use `call.value()` to make the transfer instead.

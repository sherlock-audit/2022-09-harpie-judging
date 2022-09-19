innertia
# Dangerous Methods of Transferring Admin Authority

## Summary
Incorrect addresses may be set during the transfer of admin rights.
## Vulnerability Detail
Since neither a zero address check nor two-step verification is performed during the transfer of admins' privileges, inappropriate addresses may be set.
In that case, administrative functions would not be available, and revenue would not be collected.
## Impact
Lose control of the contract, lose revenue.
## Code Snippet
```
function changeFeeController(address payable _newFeeController) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        feeController = _newFeeController;
    }
```
https://github.com/sherlock-audit/2022-09-harpie-innertiaJP/blob/6059e77610e3f595c371a34a06cbf2df6e778068/contracts/contracts/Vault.sol#L163
## Tool used

Manual Review

## Recommendation
Add a zero address check like in `Ownable.sol` of OpenZeppelin contracts.
```
function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        _transferOwnership(newOwner);
    }
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76b538c226df019a1451467d36351b3c61cd4139/contracts/access/Ownable.sol#L69

or 

two step verification would be nice.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol

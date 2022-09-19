0xSmartContract
# Instead of call(), transfer() is used for the withdraw mechanism

## Summary

```withdrawPayments``` function allows to withdraw the fees we collect in this contract. However, it does this with the ```transfer``` method, which should not be used while doing this.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

(Note that feeController is more likely to be multisign since it owns control of funds )
- The withdrawer smart contract does not implement a payable function.
- The withdrawer smart contract does implement a payable fallback which uses more than 2300 gas unit.
- The withdrawer smart contract implements a payable fallback function that needs less than 2300 gas units but is called through proxy, raising the call's gas usage above 2300.
- Using higher than 2300 gas might be mandatory for some multisig wallets.
- Gas Fees are not fixed, Ethereum has switched to POS and gas fees can change at any time, so there may be a risk of loss of funds in fixed gas fee functions such as transfers.


## Vulnerability Detail

[Vault.sol#L156-L160](https://github.com/sherlock-audit/2022-09-harpie-0xSmartContract/blob/master/contracts/contracts/Vault.sol#L156-L160)

```js
    /// @notice This function allows us to withdraw the fees we collect in this contract
    function withdrawPayments(uint256 _amount) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        require(address(this).balance >= _amount, "Cannot withdraw more than the amount in the contract.");
        feeController.transfer(_amount);
    }
```


## Tool used

Manual Review

## Recommendation
I recommend using ```call()``` instead of ```transfer()```

```js
    function withdrawPayments(uint256 _amount) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
        require(address(this).balance >= _amount, "Cannot withdraw more than the amount in the contract.");
        (bool result, ) = payable( feeController).call{value: _amount}("");
        require(result, "Failed to send Ether");
    }
```
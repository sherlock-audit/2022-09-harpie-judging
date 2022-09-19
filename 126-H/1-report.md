PwnPatrol
# Transfer failure check missing

## Summary
Transfer failure check missing

## Vulnerability Detail
Some tokens return false on transfer failure instead of reverting on transfer failure.
In such case, returned values from low-level calls should be checked for boolean values.
Currently, `transferERC20()` function checks only if low-level call did revert. It doesn't check if transfer returned false. (Should have decode `transferResult`)

This allows attacker to unsuccessfully transfer tokens (but with `transferERC20` function not reverting). Then the attacker can call `withdrawERC20()`, thus stealing funds accounted in `transferERC20`.


## Impact
Funds can be stolen form Vault contract

## Code Snippet
```javascript
function transferERC20(address _ownerAddress, address _erc20Address, uint256 _amount, uint128 _fee) public returns (bool) {
    require (msg.sender == EOA);
    (bool transferSuccess, bytes memory transferResult) = address(_erc20Address).call(
        abi.encodeWithSignature("transferFrom(address,address,uint256)", _ownerAddress, vaultAddress, _amount)
    );
    require(transferSuccess, string (transferResult));
    [...]
    return transferSuccess;
}
```


## Tool used
Manual Review

## Recommendation
Properly check return value from low-level calls
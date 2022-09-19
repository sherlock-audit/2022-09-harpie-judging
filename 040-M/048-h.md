yixxas
# batchTransferERC20() will revert if number of users is large

## Summary
`batchTransferERC20()`  will revert if number of users being protected is more than ~1500.

## Vulnerability Detail
`batchTransferERC20()` loops though all `_details()`, which are the users being protected by this protocol. Each additional call of `this.transferERC20()` cost about `20000` gas as estimated by Foundry. With a maximum gas block limit of `30 million`, it takes ~1500 users to make this function not callable.

## Impact
Once protocol is supporting more than 1500 users, the `batchTransferERC20()` will always fail, resulting in the protocol being unable to protect its users during a protocol attack. Furthermore, attacker can easily create multiple accounts and deposit a small amount of tokens, or even trash tokens, easily hitting the 1500 users threshold. 

## Code Snippet
[Transfer.sol#L108-L117](https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L108-L117)
```solidity
    function batchTransferERC20(ERC20Details[] memory _details) public returns (bool) {
        require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller.");
        for (uint256 i=0; i<_details.length; i++ ) {
            try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {}
            catch {
                emit failedERC20Transfer(_details[i].ownerAddress, _details[i].erc20Address);
            }
        }
        return true;
    }
```

## Tool used

Manual Review, Foundry

## Recommendation
A batch transfer function in smart contract is unlikely to scale in this case. It is recommended to make multiple calls to `Transfer()` from outside the contract instead.

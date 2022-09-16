CodingNameKiki
# Usage of deprecated transfer() can result in revert.

## Summary
The function withdrawPayments() is used by the Owners to withdraw the fees.

## Vulnerability Detail
transfer() uses a fixed amount of gas, which was used to prevent reentrancy. However this limit your protocol to interact with others contracts that need more than that to process the transaction.

Specifically, the withdrawal will inevitably fail when:
1.The withdrawer smart contract does not implement a payable fallback function.
2.The withdrawer smart contract implements a payable fallback function which uses more than 2300 gas units.
3.The withdrawer smart contract implements a payable fallback function which needs less than 2300 gas units but is called through a proxy that raises the call’s gas usage above 2300.

## Impact
transfer() uses a fixed amount of gas, which can result in revert.
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L159
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L156-L160

## Tool used

Manual Review

## Recommendation
Use call instead of transfer(). Example:
(bool succeeded, ) = _to.call{value: _amount}("");
require(succeeded, "Transfer failed.");
JohnSmith
# Usage of deprecated `transfer` to send ETH

## Usage of deprecated `transfer` to send ETH
### Impact
Usage of deprecated transfer on `withdrawPayments()` function call can revert.
### Proof of Concept
The original `transfer` used to send eth uses a fixed stipend 2300 gas.   This was used to prevent reentrancy.   However this limit your protocol to interact with others contracts that need more than that to process the transaction.

A good article about that:
<https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/>.

### Recommended Mitigation Steps
Used call instead.  For example

```solidity
(bool success, ) = msg.sender.call{value: amount}("");
require(success, "Transfer failed.");
```
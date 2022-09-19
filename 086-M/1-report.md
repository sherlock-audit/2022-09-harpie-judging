JohnSmith
# `changeRecipientAddress` susceptible to replay attacks

##  `changeRecipientAddress` susceptible to replay attacks

`changeRecipientAddress` can be called with same values on different date and/or different blockchain and succeed.
all transaction data will just sit there and `_signature` will always be valid for particular `_data` and `_newRecipientAddress`;
payload of this signature is very generic (two addresses) 
You assume that `msg.sender` can be compromised, so you add additional checks.
A user could  change his recipient address several times, 
Attacker can just call this function with same arguments he took from history, if he also got hold on one of recipient addresses,
avoiding 2fa or whatever you implement offchain.
I suggest adding the chainid and a salt.
Something like this: 
```diff
contracts/contracts/Vault.sol
mapping(address => uint) private salt;
-45:         require(keccak256(abi.encodePacked(msg.sender, _newRecipientAddress)) == _data, "Provided signature does not match required parameters");
+45:         require(keccak256(abi.encodePacked(msg.sender, _newRecipientAddress, block.chainid, salt[msg.sender]++)) == _data, "Provided signature does not match required parameters");

+ function getSalt(address recipient) external returns (uint) {salt[recipient];}
```
So each time this function is called `_signature` and `_data` become different for same `_newRecipientAddress` and all previous signatures become invalid for future function calls.
CodingNameKiki
# transferer address, serverSigner address and feeController address can be mistakenly set to a zero address.

## Summary
transferer address, serverSigner address and feeController address can be mistakenly set to a zero address in the constructor at a deploying time, which will result in a lot of issues.

## Vulnerability Detail
There is missing zero-address check in the constructor in Vault.sol. This way the transferer address, serverSigner address and feeController address can be mistakenly set to a zero address at a deploying time, which will lead to big issues in the contract.

1.If the transferer address is mistakenly set to a zero address, this will result into unusable function logIncomingERC20() and function logIncomingERC721(), both of the functions are used to log in transfers and only the "transferer" contract has access to them.

2.The serverSigner is an EOA responsible for providing the signature of changeRecipientAddress, if the serverSigner address is mistakenly set to a zero address, this will result in invalid signatures in the function changeRecipientAddress().

3.The feeController is an EOA that's able to only reduce the fees of users and withdraw our fees, if the feeController address is mistakenly set to a zero address, this will result into unusable function reduceERC20Fee(), function reduceERC721Fee(), function withdrawPayments() and function changeFeeController(), all of these functions check if the msg.sender is the feeController address.

## Impact
Potentially giving the initialized addresses a zero address will result into unusable functions in the Vault.sol contract.

## Code Snippet
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Vault.sol#L48-L52

## Tool used

Manual Review

## Recommendation
There should always be checks to make sure that the initialized addresses are never a zero address.
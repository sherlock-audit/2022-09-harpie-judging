HonorLt
# Possible malfunction when using try/catch.

## Summary
If the try/catch fails when logging incoming tokens, these tokens will become stuck in Vault.

## Vulnerability Detail
Transferring a token (```transferERC721```, ```transferERC20```) basically consists of 2 steps:
1) transfer the token from the user to the vault. 
2) log the incoming token.

The batch functions wrap the calls inside the ```try``` and ```catch``` meaning that it should continue the execution in case of an error. While I think it is highly unlikely with the currently known implementation of Vault, in theory, it is possible that the 1st step succeeds but the 2nd fails, leaving the token stuck in the vault forever.

A side note related to optimization is that forwarding ```400e3``` gas for a simple ERC20 transfer seems too much in my opinion:
```solidity
  try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {}
```
Especially since the fee has to be passed along meaning you should already know how much it will cost to execute the call.

## Code Snippet
ERC20 example, ERC721 is similar:
```solidity
    try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {}
            catch {
                emit failedERC20Transfer(_details[i].ownerAddress, _details[i].erc20Address);
            }
```
```solidity
  IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
        (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
        );
        require(loggingSuccess, string (loggingResult));
```

## Tool used

Manual Review

## Recommendation
Consider adding token rescue functions, e.g. if the original owner does not claim tokens for >1 year, an admin can rescue them and distribute them or keep them in the treasury.
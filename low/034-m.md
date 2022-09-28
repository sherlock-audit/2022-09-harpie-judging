Bnke0x0
# Failed transfer with low level call

## Summary
This function is utilized in a few different places in the contract. According to the [Solidity docs](https://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions)), "The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed".

## Vulnerability Detail

## Impact
Failed transfer with low level call

## Code Snippet

1. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Transfer.sol#L60-L63

                   '(bool transferSuccess, bytes memory transferResult) = address(_erc721Address).call(
            abi.encodeCall(IERC721(_erc721Address).transferFrom, (_ownerAddress, vaultAddress, _erc721Id))
        );
        require(transferSuccess, string (transferResult));'

2. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Transfer.sol#L64-L67

          '(bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC721, (_ownerAddress, _erc721Address, _erc721Id, _fee))
        );
        require(loggingSuccess, string (loggingResult));'    

3. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Transfer.sol#L98-L101

              '(bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
        );
        require(loggingSuccess, string (loggingResult));'
        

## Tool used

Manual Review

## Recommendation

Check for contract existence on low-level calls, so that failures are not missed.

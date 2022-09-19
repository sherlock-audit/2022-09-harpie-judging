0xSmartContract
# Using incompatible Solidity version

## Summary
The transfer.sol contract uses the ```abi.encodeCall``` member and the pragma version is ^0.8. However, ```abi.encodeCall``` was introduced with solidity 0.8.11. Therefore, it will not work with compilers between 0.8.0 > 0.8.11.



https://blog.soliditylang.org/2021/12/20/solidity-0.8.11-release-announcement/

## Vulnerability Detail

[Transfer.sol#L57-L70](https://github.com/sherlock-audit/2022-09-harpie-0xSmartContract/blob/master/contracts/contracts/Transfer.sol#L57-L70)

```js
    function transferERC721(address _ownerAddress, address _erc721Address, uint256 _erc721Id, uint128 _fee) public returns (bool) {
        require(_transferEOAs[msg.sender] == true || msg.sender == address(this), "Caller must be an approved caller.");
        require(_erc721Address != address(this));
        (bool transferSuccess, bytes memory transferResult) = address(_erc721Address).call(
            abi.encodeCall(IERC721(_erc721Address).transferFrom, (_ownerAddress, vaultAddress, _erc721Id))
        );
        require(transferSuccess, string (transferResult));
        (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC721, (_ownerAddress, _erc721Address, _erc721Id, _fee))
        );
        require(loggingSuccess, string (loggingResult));
        emit successfulERC721Transfer(_ownerAddress, _erc721Address, _erc721Id);
        return transferSuccess;
    }
```
https://github.com/ethereum/solidity/releases/tag/v0.8.11

It is critical to use up-to-date version for Security-related agreements such as this one.

## Tool used
Manual Review

## Recommendation
Use pragma 0.8.11

Tomo
# Call to non-existing contracts returns success

## 💥 Impact

As written in the [solidity documentation](https://github.com/sherlock-audit/2022-09-harpie-Tomosuke0930/issues/%5B%3Chttps://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions%3E%5D(%3Chttps://docs.soliditylang.org/en/develop/control-structures.html#error-handling-assert-require-revert-and-exceptions%3E)), the low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed

## 🕵️‍♂️ Vulnerability Detail

In `transferERC20` and `transferERC721` functions, using the low-level function but there is no check for contract existence.

This leads to the loss of the user fund because the result will be returned true when the destination address is none.

## 📝 Code Snippet

There is no check the `vaultAddress` is not `address(0)`

Therefore, the `vaultAddress` can be `address(0).`

If the deployer mistakenly set this value to `address(0)`, two functions don’t return revert.

```solidity
constructor(address _vaultAddress, address _transferEOASetter) {
        vaultAddress = _vaultAddress;
        transferEOASetter = _transferEOASetter;
}

function transferERC20(/* ... */) public returns (bool) {
    /* ... */
    (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
         abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
    );
    /* ... */
}

function transferERC721(/* ... */) public returns (bool) {
    /* ... */
    (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
         abi.encodeCall(Vault.logIncomingERC721, (_ownerAddress, _erc721Address, balance, _fee))
    );
    /* ... */
}
```

1. As a reference, OpenZeppelin's `Address.functionCall()` will check and `require(isContract(target), "Address: call to non-contract");`

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.5.0/contracts/utils/Address.sol#L135](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.5.0/contracts/utils/Address.sol#L135)

```solidity
function functionCallWithValue(
    address target,
    bytes memory data,
    uint256 value,
    string memory errorMessage
) internal returns (bytes memory) {
    require(address(*this*).balance >= value, "Address: insufficient balance for call");
    require(isContract(target), "Address: call to non-contract");

    (bool success, bytes memory returndata) = target.call{value: value}(data);
    return verifyCallResult(success, returndata, errorMessage);
}
```

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.5.0/contracts/utils/Address.sol#L36-L42](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.5.0/contracts/utils/Address.sol#L36-L42)

```solidity
function isContract(address account) internal view returns (bool) {
        // This method relies on extcodesize/address.code.length, which returns 0
        // for contracts in construction, since the code is only stored at the end
        // of the constructor execution.

        return account.code.length > 0;
    }
```

## 🚜 Tools Used

Manual Review

## ✅ Recommendation

You should check the `vaultAddress` is not `address(0)`.

```solidity
// before
constructor(address _vaultAddress, address _transferEOASetter) {
    vaultAddress = _vaultAddress;
    transferEOASetter = _transferEOASetter;
    }
// after
constructor(address _vaultAddress, address _transferEOASetter) {
    require(vaulAddress != address(0), "CAN'T BE ADDRESS ZERO");
    vaultAddress = _vaultAddress;
    transferEOASetter = _transferEOASetter;
    }
```

## 👬 Similar Issue

[https://github.com/code-423n4/2022-03-rolla-findings/issues/46](https://github.com/code-423n4/2022-03-rolla-findings/issues/46)
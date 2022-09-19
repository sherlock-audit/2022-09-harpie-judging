defsec
# Fee is directly get by user argument


## Summary

On the protocol, Fee is get by an user argument. However, there is no bound on the fee.  Even if there is no loss on the protocol user funds, the users should be paid necessary fee to join to protocol.  

## Vulnerability Detail

During the code review, It has been observed that fee does not have any lower bound on the protocol. 

## Impact

The protocol can get lower fee than expected.

## Code Snippet

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L88

```
    function transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool) {
        require (_transferEOAs[msg.sender] == true || msg.sender == address(this), "Caller must be an approved caller.");
        require(_erc20Address != address(this));
        // Do the functions after the following line occur if the following line fails? Does it revert? Test
        uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
        IERC20(_erc20Address).safeTransferFrom(
            _ownerAddress, 
            vaultAddress, 
            balance
        );
        (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
            abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
        );
        require(loggingSuccess, string (loggingResult));
        emit successfulERC20Transfer(_ownerAddress, _erc20Address);
        return loggingSuccess;
    }
```

## Tool used

Manual Review

## Recommendation

Define bound on the protocol fee. 
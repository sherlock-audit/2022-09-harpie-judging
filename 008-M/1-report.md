minhquanym

medium

# There is no limit on the amount of fee users have to pay

## Summary

https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L57
https://github.com/Harpieio/contracts/blob/97083d7ce8ae9d85e29a139b1e981464ff92b89e/contracts/Transfer.sol#L88

## Vulnerability Detail

There is no upper limit on the amount of fee users have to pay to withdraw their funds back. So any EOA can call transfer function on `Transfer` contract can set an unreasonable amount of fee and users have to pay it if they want their funds back. We need to make sure that users' funds cannot be loss even when the protocol acts maliciously. 

## Impact

In case the protocol acts maliciously and set `fee = 1e18` to transfer users' fund to `Vault`, users cannot withdraw their funds since fee is too high.

## Proof of Concept

In both `transferERC20()` and `transferERC721()`, EOA is caller and can set `fee` param to any value it wants. 
```solidity
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

And users need to send enough fee (native token) to withdraw their fund back on `Vault`
```solidity
function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {
      require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
      require(_erc721Address != address(this), "The vault is not a token address");
      require(canWithdrawERC721(_originalAddress, _erc721Address, _id), "Insufficient withdrawal allowance.");
      require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");

      _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].isStored = false;
      _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee = 0;
      IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
}
```

## Tool used

Manual Review

## Recommendation

Consider adding an upper limit on the amount of fee users need to pay

## Lead Senior Watson

Currently there is no way to revoke a change fee controller request. I'd shy away from using a mapping, adds unnecessary overhead when it can be handled by a `pendingFeeController` variable. Also important to note that mapping in `changeFeeController()` is not cleared.

## Harpie Team

Using leastwood's suggestion of a timelock for feeController. Fix [here](https://github.com/Harpieio/contracts/pull/4/commits/9b75a000f6cb0798e650f1433012b2b52f7a0e2b). Supplementary fixes for this issue: 
[1](https://github.com/Harpieio/contracts/pull/4/commits/c60dc166aab6f7067379ea3f1e39be2ae17cc2dc), 
[2](https://github.com/Harpieio/contracts/pull/4/commits/ea97548c379ec9b48e42724a52a1ee7bd4cce6b7), 
[3](https://github.com/Harpieio/contracts/pull/4/commits/8cfc07577c49eb0b0713fb5499ea9313153c2c7c). 

## Lead Senior Watson

Confirmed fixes. 

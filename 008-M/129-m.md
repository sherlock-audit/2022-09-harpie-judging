HonorLt
# Unbounded fee

## Summary
The transferrer can choose an arbitrary ```_fee```.

## Vulnerability Detail
There are no validations against the ```_fee```, it is left up for the caller (```_transferEOAs```) to decide.
If the fee is too high, the tokens will be kept in the vault as hostages. It should be profitable after the fees to make the withdrawal.

There are too many risks, e.g. if the transfer EOA is compromised or if an off-chain script mistake inputs the wrong number, this could lead to unrecoverable assets for the end users.

Of course, the fee can be reduced manually later, but I believe the users would feel safer if the protocol had some precautions upfront.

## Code Snippet
```solidity
 function transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool)
 function transferERC721(address _ownerAddress, address _erc721Address, uint256 _erc721Id, uint128 _fee) public returns (bool)
```
```solidity  
  _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee += _fee;
```
```solidity   
   _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee += _fee;
```

## Tool used

Manual Review

## Recommendation
Consider introducing a parameter of max fee that the user is willing to pay per transfer or introduce a global upper boundary that should not be exceeded.

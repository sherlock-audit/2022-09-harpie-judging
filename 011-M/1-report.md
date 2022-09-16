millers.planet
# Possible lock of assets

## Summary

Lack of check for ERC721Receiver implementation.

## Vulnerability Detail

On `Vault.sol`, the `withdrawERC721()` function makes an ERC-721 transfer to the `msg.sender` of the transaction on line 137:
`IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);`

While the protocol allows smart contracts to receive the tokens if the caller of the transaction is a smart contract, it does not check that it implements the IERC721-Receiver.(https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#IERC721Receiver)

## Impact

If the address for `_recipientAddress[_originalAddress]` (custodial) is a smart contract  that does not implement the IERC721Receiver interface, the ERC721 tokens could be locked on the smart contract and lost forever.

## Code Snippet
```
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

https://github.com/sherlock-audit/2022-09-harpie-JuanXavier/blob/master/contracts/contracts/Vault.sol#L137

## Tool used

Manual Review

## Recommendation

Implement the `safeTransferFrom()` from the IERC721 interface to make sure that in case the recipient for this transfer is a contract, it is able to receive the ERC721 tokens without locking them.
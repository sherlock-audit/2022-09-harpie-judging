cccz
# Using the transferFrom function of an ERC721 contract may freeze the user's NFT

## Summary
Using the transferFrom function of an ERC721 contract may freeze the user's NFT
## Vulnerability Detail
When using the transferFrom function of an ERC721 contract to send an NFT, if the receiving address is a smart contract and does not support ERC721, the NFT can be frozen in the contract.
## Impact
the NFT can be frozen in the contract.
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
## Tool used

Manual Review

## Recommendation
Use the ERC721 contract's safeTransferFrom function to send NFTs

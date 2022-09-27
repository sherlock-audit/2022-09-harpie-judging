Tomo
# Use `safeTransferFrom` for ERC721

## 💥 Impact

Users may be lost NFT

## 🕵️‍♂️ Vulnerability Detail

The `withdrawERC721` function transfer NFT to `msg.sender` .

However `msg.sender` may be the contracts that cannot handle them, and therefore the token will be locked forever.

As per the documentation of EIP-721:

> A wallet/broker/auction application MUST implement the wallet interface if it will accept safe transfers.

Ref: [https://eips.ethereum.org/EIPS/eip-721](https://eips.ethereum.org/EIPS/eip-721%5D(https://eips.ethereum.org/EIPS/eip-721))

## 📝 Code Snippet

```solidity
function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {
				/* ... */
        IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
    }
```

## 🚜 Tools Used

Manual

## ✅ Recommendation

Use `safeTransferFrom` instead of `transferFrom`

```solidity
// before
function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {
				/* ... */
        IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
    }
// after
function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {
				/* ... */
        IERC721(_erc721Address).safeTransferFrom(address(this), msg.sender, _id);
    }
```

## 👬 Similar Issue

[https://github.com/code-423n4/2022-04-backed-findings/issues/83](https://github.com/code-423n4/2022-04-backed-findings/issues/83)
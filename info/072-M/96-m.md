Tomo
# Reentrancy in `withdrawERC20` function

## 💥 Impact

The `withdrawERC20` function makes an external call to the `msg.sender` by way of `_safeTransfer`. This allows the caller to reenter this and other functions in this and other protocol files.

## 🕵️‍♂️ Vulnerability Detail

This is `SafeERC20.sol` of Openzeppelin.

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol)

```solidity
function safeTransfer(
        IERC20 token,
        address to,
        uint256 value
    ) internal {
        _callOptionalReturn(token, abi.encodeWithSelector(token.transfer.selector, to, value));
    }
function _callOptionalReturn(IERC20 token, bytes memory data) private {
        // We need to perform a low level call here, to bypass Solidity's return data size checking mechanism, since
        // we're implementing it ourselves. We use {Address.functionCall} to perform this call, which verifies that
        // the target address contains contract code and also asserts for success in the low-level call.

        bytes memory returndata = address(token).functionCall(data, "SafeERC20: low-level call failed");
        if (returndata.length > 0) {
            // Return data is optional
            require(abi.decode(returndata, (bool)), "SafeERC20: ERC20 operation did not succeed");
        }
    }
```

## 📝 Code Snippet

```solidity
function withdrawERC20(address _originalAddress, address _erc20Address) payable external {
				/* ... */
        IERC20(_erc20Address).safeTransfer(msg.sender, amount);
    }
```

## 🚜 Tools Used

Manual

## ✅ Recommendation

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol)

Add `nonreentrant` modifier to `withdrawERC20` function.

## 👬 Similar Issue

[https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/80](https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/80)
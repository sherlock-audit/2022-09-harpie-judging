Tomo
# Unsupported fee-on-transfer tokens

## 💥 Impact

The discrepancy would break the internal accounting system of the contract.

## 🕵️‍♂️ Vulnerability Detail

Some ERC20 tokens(e.g. `STA`, `PAXG`,in the future USDC,USDT), allow for charging a fee any time transfer() or transferFrom() is called.

For more detail, please read this.

[https://github.com/d-xo/weird-erc20#fee-on-transfer](https://github.com/d-xo/weird-erc20#fee-on-transfer](https://github.com/d-xo/weird-erc20%23fee-on-transfer))

### *Exploit Scenario*

1. The `STA` token is a deflation token that charges 1% fee for every transfer.
2. If `transferERC20address(owner),address(STA),unit(fee)` is executed, and 10000 STA will be sent to the contract and each value will be as follows.

```solidity
// Transfer.sol
function transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool) {
    /* ... */
    // balance will be 10000 but, actually 9900
    uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress); 
    IERC20(_erc20Address).safeTransferFrom( /* ... */,balance);
    (/* ... */) = address(vaultAddress).call(/* ... */ ,balance,));
}

// Vault.sol
function logIncomingERC20(address _originalAddress, address _erc20Address, uint256 _amount, uint128 _fee) external{
    /* ... */
    // .amountStored will be 10000 but, actually 9900
    _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount); 
}
```
3. However, the contracts will only receive 9900 STA, thus only 100 STA is escrowed in the contract. 
4. When executing `withdrawERC20` to withdraw `STA` token in Vault, the `amount` will be 10000.
5. Therefore, this transaction will return revert because of insufficient of tokens in this contract. Users cannot withdraw the tokens they have transferred to vault.
6. Also, if other users deposit `STA` tokens in this contract, leads to a loss of other users’ funds because users can withdraw more tokens than they actually transferred 

```solidity
function withdrawERC20(address _originalAddress, address _erc20Address) payable external {
		/* ... */
    uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
		/* ... */
    IERC20(_erc20Address).safeTransfer(msg.sender, amount);
}

function canWithdrawERC20(address _originalAddress, address _erc20Address) public view returns (uint256) {
    return _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored;
}
```

## 📝 Code Snippet

```solidity
// Transfer.sol
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

// Vault.sol
function logIncomingERC20(address _originalAddress, address _erc20Address, uint256 _amount, uint128 _fee) external{
        require(msg.sender == transferer, "Only the transferer contract can log funds.");
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee += _fee;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
    }

function canWithdrawERC20(address _originalAddress, address _erc20Address) public view returns (uint256) {
        return _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored;
    }

function withdrawERC20(address _originalAddress, address _erc20Address) payable external {
        require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
        require(_erc20Address != address(this), "The vault is not a token address");
        require(canWithdrawERC20(_originalAddress, _erc20Address) > 0, "No withdrawal allowance.");
        require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
        uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored = 0;
        _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = 0;
        IERC20(_erc20Address).safeTransfer(msg.sender, amount);
    }
```

## 🚜 Tools Used

Manual

## ✅ Recommendation

Ensure that to check previous balance/after balance equals to amount for any rebasing/inflation/deflation.

Create a whitelist in contracts to restrict token addresses.

## 👬 Similar Issue

[https://github.com/code-423n4/2022-04-axelar-findings/issues/5](https://github.com/code-423n4/2022-04-axelar-findings/issues/5%5D(https://github.com/code-423n4/2022-04-axelar-findings/issues/5))

[https://github.com/code-423n4/2022-02-concur-findings/issues/158](https://github.com/code-423n4/2022-02-concur-findings/issues/158%5D(https://github.com/code-423n4/2022-02-concur-findings/issues/158))

[https://github.com/code-423n4/2022-04-phuture-findings/issues/43](https://github.com/code-423n4/2022-04-phuture-findings/issues/43%5D(https://github.com/code-423n4/2022-04-phuture-findings/issues/43))

### 🧑‍⚖️👩‍⚖️ For judges

[https://github.com/code-423n4/org/issues/3#issuecomment-1055616192](https://github.com/code-423n4/org/issues/3#issuecomment-1055616192](https://github.com/code-423n4/org/issues/3%23issuecomment-1055616192))
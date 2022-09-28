hansfriese
# Funds might be locked inside the `Vault` for the fee-on-transfer tokens.

## Summary
Funds might be locked inside the `Vault` for the fee-on-transfer tokens.


## Vulnerability Detail
```
File: 2022-09-harpie-hansfriese\contracts\contracts\Transfer.sol
092:         uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
093:         IERC20(_erc20Address).safeTransferFrom(
094:             _ownerAddress, 
095:             vaultAddress, 
096:             balance
097:         );
098:         (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
099:             abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
100:         );
101:         require(loggingSuccess, string (loggingResult));
```

## Impact
Some weird ERC20 tokens are fee-on-transfer tokens and the `Vault` might save more amount of tokens than it received.

In this case, users can't withdraw their token forever because it reverts with insufficient funds.


## Proof of Concept
As we can see [here](https://github.com/d-xo/weird-erc20#fee-on-transfer), there are some fee-on-transfer tokens.

So the below scenario would be possible.

- ERC20 token `T` is a fee-on-transfer token.
- `Transfer` contract transfers 100 wei of `T` from Alice using `Transfer.transferERC20()`.
- `Vault` contract received 99 wei of `T` but `_erc20WithdrawalAllowances` for Alice will be 100 wei.
```
File: 2022-09-harpie-hansfriese\contracts\contracts\Transfer.sol
098:         (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
099:             abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
100:         );
101:         require(loggingSuccess, string (loggingResult));
```

```
File: 2022-09-harpie-hansfriese\contracts\contracts\Vault.sol
82:     function logIncomingERC20(address _originalAddress, address _erc20Address, uint256 _amount, uint128 _fee) external{
83:         require(msg.sender == transferer, "Only the transferer contract can log funds.");
84:         _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee += _fee;
85:         _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored += uint128(_amount);
86:     }
87: 
```

- When Alice tries to withdraw `T`, there is no option to withdraw some part of the total amount. So it will revert when it tries to withdraw 100 wei of `T` with insufficient funds.

```
File: 2022-09-harpie-hansfriese\contracts\contracts\Vault.sol
118:     function withdrawERC20(address _originalAddress, address _erc20Address) payable external {
119:         require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
120:         require(_erc20Address != address(this), "The vault is not a token address");
121:         require(canWithdrawERC20(_originalAddress, _erc20Address) > 0, "No withdrawal allowance.");
122:         require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
123: 
124:         uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
125:         _erc20WithdrawalAllowances[_originalAddress][_erc20Address].amountStored = 0;
126:         _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee = 0;
127:         IERC20(_erc20Address).safeTransfer(msg.sender, amount);
128:     }
```

## Tool used
Manual Review


## Recommendation
`Transfer.transferERC20()` should be modified like below.

```
function transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool) {
    require (_transferEOAs[msg.sender] == true || msg.sender == address(this), "Caller must be an approved caller.");
    require(_erc20Address != address(this));
    // Do the functions after the following line occur if the following line fails? Does it revert? Test

    uint256 beforeBalance = IERC20(_erc20Address).balanceOf(vaultAddress); //+++++++++++++++++

    uint256 balance = IERC20(_erc20Address).balanceOf(_ownerAddress);
    IERC20(_erc20Address).safeTransferFrom(
        _ownerAddress, 
        vaultAddress, 
        balance
    );

    balance = IERC20(_erc20Address).balanceOf(vaultAddress) - beforeBalance; //+++++++++++++++++

    (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
        abi.encodeCall(Vault.logIncomingERC20, (_ownerAddress, _erc20Address, balance, _fee))
    );
    require(loggingSuccess, string (loggingResult));
    emit successfulERC20Transfer(_ownerAddress, _erc20Address);
    return loggingSuccess;
}
```
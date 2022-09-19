xiaoming90
# Withdraw Functions Did Not Return Excess ETH

## Summary

Excess ETH is not returned to the users within the withdraw function, thus causing users to lose their funds.

## Vulnerability Detail

If users send excess ETH to the withdraw functions for the fee, the difference will not be refunded to the user.

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L118

```
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

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L129

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

## Impact

Loss of funds for the users as the excess ETH is not refunded to them

## Recommendation

Consider refunding the difference or ensure that `msg.value == fee`
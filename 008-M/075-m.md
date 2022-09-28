xiaoming90
# User's Assets Can Be Locked If Admin Is Compromised Or Turned Rouge

## Impact

All users' assets will be locked if the admin's private keys are compromised or the admin turns rouge.

## Vulnerability Detail

Based on the current design of Harpie firewall, there is no way for the admin to withdraw users' assets from the `Vault`. Therefore, the users can rest assured that their assets will not be withdrawn by a malicious or compromised admin under any circumstance.

However, it was observed that it is possible for a malicious or compromised admin to "lock" users' assets within the vault. If the private keys are compromised or an admin turns rouge, the attacker could call the `transferERC20` or `transferERC721` function against all Harpie users, causing their assets to be transferred to the Vault. Since the `_fee` parameter is unbounded, the attacker calls the `transferERC20` or `transferERC721` function with the `_fee` set to a large value (e.g. 1,000,000,000 ETH).

As a result, even if the affected users could technically withdraw their assets from the Vault, no one would be willing to pay such a large fee to withdraw their assets. This is because their assets are likely to be worth less than the fee (e.g. 1,000,000,000 ETH). This effectively locked users' assets.

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Transfer.sol#L88

```solidity
/// @notice This function transfers ERC20s to a noncustodial vault contract.
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

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Transfer.sol#L88

```solidity
/// @notice This function transfers ERC721s to a noncustodial vault contract.
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

#### Can calling the `reduceERC20Fee` and `reduceERC721Fee` functions solves the issue?

Technically, the Harpie admin can call the `reduceERC20Fee` and `reduceERC721Fee` functions to change the fee from 1,000,000,000 ETH back to the normal value, and the users can withdraw the assets as usual.

However, a sophisticated attacker would first call the `changeFeeController` to change the `feeController` to point to the attacker's EOA address before executing the transfers/attacks to prevent Sentiment admins from calling  `reduceERC20Fee` and `reduceERC721Fee` functions.

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L163

```solidity
/// @notice This function allows us to change the signer that we use to reduce and withdraw fees
function changeFeeController(address payable _newFeeController) external {
    require(msg.sender == feeController, "msg.sender must be feeController.");
    feeController = _newFeeController;
}
```

Once the attacker has changed the `feeController` to point to their own EOA, Harpie will not be able to call the `reduceERC20Fee` and `reduceERC721Fee` functions anymore.

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L141

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L148

```solidity
/// @notice These functions allow Harpie to reduce (but never increase) the fee upon a user
function reduceERC20Fee(address _originalAddress, address _erc20Address, uint128 _reduceBy) external returns (uint128) {
    require(msg.sender == feeController, "msg.sender must be feeController.");
    require(_erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee >= _reduceBy, "You cannot reduce more than the current fee.");
    _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee -= _reduceBy;
    return _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee;
}

function reduceERC721Fee(address _originalAddress, address _erc721Address, uint128 _id, uint128 _reduceBy) external returns (uint128) {
    require(msg.sender == feeController, "msg.sender must be feeController.");
    require(_erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee >= _reduceBy, "You cannot reduce more than the current fee.");
    _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee -= _reduceBy;
    return _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee;
}
```

In addition, only the attacker can withdraw the fee accrued in the vault. Therefore, attempts to ask the users to pay the fee first and refund the fee back to the users later will also not work.

https://github.com/sherlock-audit/2022-09-harpie-xiaoming9090/blob/49ba081b51a4c986421ed2c1f9e5340b90c04f6d/contracts/contracts/Vault.sol#L156

```solidity
/// @notice This function allows us to withdraw the fees we collect in this contract
function withdrawPayments(uint256 _amount) external {
    require(msg.sender == feeController, "msg.sender must be feeController.");
    require(address(this).balance >= _amount, "Cannot withdraw more than the amount in the contract.");
    feeController.transfer(_amount);
}
```

## Impact

All the assets within the wallets of Harpie users will be locked in the Vault. Since the contract is non-upgradable, the assets will be locked forever if this incident occurs.

The probability of the private keys of the admin getting compromised or the admin turning rouge is low, but the impact of this issue is extremely high. Thus, I'm marking this issue as Medium.

Although the probability is low, many security incidents in the past were caused by compromised private keys and internal team member turning rouge (e.g. Ronin Bridge Hack), resulting in massive loss to the users and community. Therefore, it is prudent to mitigate or reduce the risk of this issue.

## Recommendation

Per the [Haripe documentation](https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/pricing), Haripe will charge a 7% fee on any asset that we successfully recover. Therefore, within the `transferERC20` and `transferERC721` functions, implement an additional check to revert the transaction if the `fee` is more than 7%.

Note that having server-side validation (off-chain) is not sufficient. On-chain validation within the contracts should also be in place for defense in depth.
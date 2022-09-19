0xSmartContract
# ERC20 Event-Emit important information missing

## Summary
Event is an inheritable member of a contract. An event is emitted, it stores the arguments passed in transaction logs. These logs are stored on blockchain and are accessible using address of the contract till the contract is present on the blockchain.

There is no token number in Event-Emits spread after successful transfers of ERC20 and ERC721.

It is natural that there is no number information in ERC721, but the information of how many tokens are transferred in ERC20 is important, it should be added


## Impact

1 - Alice use Harpie Service

2- 10000 UNI tokens in Alice's EOA Wallet were hacked and the Harpie Service intervened in part of the attack and 70.000 UNI tokens were recovered

2- Due to the result followed by the event records, the recovery was recorded as successful in the web interface, but only some of the tokens were recovered, the recovery rate could be even lower.


## Code Snippet

```js

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

## Tool used

Manual Review

## Recommendation

```js

    /// @dev These events fire when a token transfer and subsequent logging is successful

    event successfulERC20Transfer(address indexed ownerAddress , address indexed erc20Address, uint256 amountsuccess);

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
        emit successfulERC20Transfer(_ownerAddress, _erc20Address, balance);
        return loggingSuccess;
    }

```
saian
# `changeFeeController()` lacks 2 step address transfer

## Vulnerability details

If the `feeController` address is accidentally set to a wrong address, fees paid will be stuck in the contract as there is no other way to withdraw the fees or update the address,
Functions `reduceERC20Fee` and `reduceERC721Fee` cannot be executed to reduce fee of the withdraw allowance
Since transfer functions allows admin to set a arbitrary fee amount, any accidental transfer with high fee amount or multiple execution of `transferERC20` with 0 token amount will set a high fee. Since fees cannot be reduced, users will have to pay a high fee, or will be unable to withdraw their assets from the vault

## Code Snippet

https://github.com/sherlock-audit/2022-09-harpie-saianmk/blob/924f1e416faa108d28b798d52b3308bcb9f27093/contracts/contracts/Transfer.sol#L120

```
    function setTransferEOA(address _newTransferEOA, bool _value) public {
        require(msg.sender == transferEOASetter, "Caller must be an approved caller.");
        _transferEOAs[_newTransferEOA] = _value;
    }
```

https://github.com/sherlock-audit/2022-09-harpie-saianmk/blob/924f1e416faa108d28b798d52b3308bcb9f27093/contracts/contracts/Vault.sol#L157

```
    function withdrawPayments(uint256 _amount) external {
        require(msg.sender == feeController, "msg.sender must be feeController.");
```

https://github.com/sherlock-audit/2022-09-harpie-saianmk/blob/924f1e416faa108d28b798d52b3308bcb9f27093/contracts/contracts/Vault.sol#L142

```
    function reduceERC20Fee(address _originalAddress, address _erc20Address, uint128 _reduceBy) external returns (uint128) {
        require(msg.sender == feeController, "msg.sender must be feeController.");
```

https://github.com/sherlock-audit/2022-09-harpie-saianmk/blob/924f1e416faa108d28b798d52b3308bcb9f27093/contracts/contracts/Vault.sol#L149

```
    function reduceERC721Fee(address _originalAddress, address _erc721Address, uint128 _id, uint128 _reduceBy) external returns (uint128) {
        require(msg.sender == feeController, "msg.sender must be feeController.");
```

## Tool used

Manual Review

## Recommendation

Implement a 2 step address change
1. Approve a new address as pendingFeeController
2. Pending address claims the feeController 

Add input validation and limits for fee value

Revert transfer if user's token balance is zero

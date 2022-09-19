# Preface

Submitting a report for completeness / value-added service. There are a number of low-severity issues that are worth changing for simplicity and elegance, such as the `.call()` to direct function calls.

# Low Severity Findings

## L01: Replace `.call()` with direct function calls

### Description

The vault’s logging methods and erc721’s `transferFrom()` are called indirectly.

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L60-L67
https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L98-L101

Apart of reduced readability, the revert reason becomes muddled. If we take `logIncomingERC20()` as an example, the only way for the function to revert is:

1. `transferer` ≠ Transfer contract (very unlikely to happen)

2. Addition overflow, which reverts with `reverted with panic code 0x11 (Arithmetic operation underflowed or overflowed outside of an unchecked block)`

For the second scenario, we see that the [error returned is `NH{q`, which isn’t a human readable error](https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/commit/1c6c96e6a4e2ad72d948115548d52cd78aba59a2).

### Recommendation

Simply change to direct calls. Not only is it cleaner, the error message(s) should be straightforward as well.

```solidity
// maybe save vaultAddress as type Vault to avoid casting
Vault(vaultAddress).logIncomingERC20(_ownerAddress, _erc20Address, balance, _fee);
IERC721(_erc721Address).transferFrom(_ownerAddress, vaultAddress, _erc721Id);
Vault(vaultAddress).logIncomingERC721(_ownerAddress, _erc721Address, _erc721Id, _fee);
```

## L02: Misleading function name `canWithdrawERC20`

### Description

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/1c6c96e6a4e2ad72d948115548d52cd78aba59a2/contracts/contracts/Vault.sol#L96-L98

This function returns the ERC20 amount withdrawable by the user, but the function name implies a check on whether the user can withdraw a specified ERC20 token. A better name would be `erc20AmountWithdrawable()`.

## L03: Make `private` variables `public`

The `private` variables `vaultAddress`, `transferEOASetter`, `_transferEOAs`, `transferer`, `serverSigner`, `feeController` and `_recipientAddress` are better off being public variables. There are multiple reasons for doing so:

1. Checking deployment: Helps to ensure that the variables are correctly set after contract deployment, especially `vaultAddress` in `Transfer` and `transferer` in `Vault`. The current implementation makes spotting misconfigurations difficult.

2. Ease of Debugging and state reading: For dev operations, the team might want to check whether an address’s transfer permissions has been revoked. This is difficult at the moment, because the `_transferEOAs` mapping isn’t exposed.

3. Transparency: Related to state reading. Users are able to clearly see who the `feeController` is (Eg. it is supposed to be a multisig). Users need not trust the protocol; they can verify for themselves.

## L04: Missing revert messages

### Description

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/1c6c96e6a4e2ad72d948115548d52cd78aba59a2/contracts/contracts/Transfer.sol#L59
https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/1c6c96e6a4e2ad72d948115548d52cd78aba59a2/contracts/contracts/Transfer.sol#L90

The lack of a revert reason makes debugging difficult.

## L05: Use latest solidity version

### Description

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/1c6c96e6a4e2ad72d948115548d52cd78aba59a2/contracts/hardhat.config.js#L11

`sol 0.8.13` was used for contract compilation. Recent releases contain important bugfixes. It is advisable to use the latest `0.8.17` version.

# Non-Critical Findings

## NC01: `withdrawPayments()` to withdraw full amount

### Description

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/1c6c96e6a4e2ad72d948115548d52cd78aba59a2/contracts/contracts/Vault.sol#L156-L160
For ease of use, `withdrawPayments()` can simply withdraw the entire ETH balance in the Vault. The `_amount` parameter feels unnecessary (why would partial withdrawals be performed?)

```solidity
function withdrawPayments() external {
  require(msg.sender == feeController, "msg.sender must be feeController");
  feeController.transfer(address(this).balance);
}
```

## NC02: Optimised for-loop

### Line References

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L76-L83

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L110-L115

### Description

The for-loop iterations can be optimised to be as such:

```solidity
uint256 detailsLength = _details.length;
for (uint256 i; i < detailsLength;) {
   ...
   unchecked { ++i; }
}
```

## NC03: Skip `== true` and `== false` checks

### Line References

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L58

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L75

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L89

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Transfer.sol#L109

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Vault.sol#L70

### Description

It’s a tad more efficient to skip checking expressions that evaluate to booleans against `true` / `false`.

```diff
- require(_transferEOAs[msg.sender] == true || msg.sender == address(this), "");
+ require(_transferEOAs[msg.sender] || msg.sender == address(this), "");

- require(_isChangeRecipientMessageConsumed[data] == false, "Already used this signature");
+ require(!_isChangeRecipientMessageConsumed[data], "Already used this signature");
```

## NC04: Fee subtraction can be unchecked

### Description

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Vault.sol#L143-L144

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/83f321b95cfbf44514f5b88f98bf1576f5c3dc8c/contracts/contracts/Vault.sol#L150-L151

The fee subtractions can be in an unchecked block to save gas, since the condition has been checked in the line above.

```solidity
unchecked { _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee -= _reduceBy; }

unchecked { _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee -= _reduceBy; }
```

## NC05: Turn on optimization + solc runs + IR flag

### Description

The contracts are currently compiled without optimization. Help users save gas by [turning on optimization, turning up the number of solc runs and use the IR optimizer](https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/commit/d6405c619d8417e9e5d90227dae418b0a83f8869).

## NC06: Avoid additional SLOAD by shifting cached value instantiation

### Description

https://github.com/sherlock-audit/2022-09-harpie-hickuphh3/blob/d6405c619d8417e9e5d90227dae418b0a83f8869/contracts/contracts/Vault.sol#L121-L124

Shift the line `uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);` to before the require statement and use the cached value to avoid doing an additional SLOAD.

```diff
+ uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
- require(canWithdrawERC20(_originalAddress, _erc20Address) > 0, "No withdrawal allowance.");
+ require(amount > 0, "No withdrawal allowance.");
require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");

- uint256 amount = canWithdrawERC20(_originalAddress, _erc20Address);
```

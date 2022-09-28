__141345__
# Intentionally transfer failure to steal the gas

## Summary

Some tokens are not transferrable, which could be abused to trick the transferEOAs to front run the transaction with high gas. The token transfer will fail but gas will be consumed. Eventually steal the transferEOAs' fund by unbounded gas cost. 

The miners have the incentive to perform this kind of attack, since they will directly benefit from the high gas fee. This can also be used to grief the system, because the cost of the attackers is low, just the lowest gas fee for one single transfer.

The best way to mitigate is a whitelist for supported assets.


## Vulnerability Detail

Some users may want to trick the transferEOAs to front run with high gas fee, but at the end transfer no tokens. Some malicious ERC20/ERC721 token reverts on every transfer could be like this:
```solidity
    function transfer(address dst, uint256 amount) external {
        consumeGas();
        revert();
    }

    function transferFrom(address src, address dst, uint256 amount) external {
        consumeGas();
        revert();
    }
```

The miners benefit directly from high gas. To perform the attack, the aim is to get the gas fee as high as possible, batch transfer would be called, and the input array would be extremely long. Even worse, the malicious token can have a `while()` in the `transfer()/transferFrom()` to consume all the 400e3 gas provided.

The attacker only need to send one transfer to untrusted address with the lowest gas fee, the `Transfer` contract will be triggered. Hence the cost for this attack is very low.

Some existing tokens will fail the transfer for sure, such as:
- some debt token (AAVE)
- compound cToken with borrow balance
- some token with pause mode (USDT)

In these cases, the transfer will fail anyway, no token will be transferred to the vault, but wasting the high gas fee from transferEOAs.


## Impact

The `Transfer` contract front run mechanism will be abused, transferEOAs will continuously loss fund due to high gas. Miners benefit from the high gas fee have the motivation for this attack. Griefing attack could also happen to influence the contract normal function. During chain congestion, the gas price will be much higher than normal times, making the loss even bigger.


## Code Snippet

- AAVE debt token 
can not transfer
```solidity
  function transfer(,) public virtual override returns (bool) {
    // ...
    revert('TRANSFER_NOT_SUPPORTED');
  }

  function transferFrom(,,) public virtual override returns (bool) {
    // ...
    revert('TRANSFER_NOT_SUPPORTED');
  }
```

- Compound cToken 
can not transfer all balance if the user has borrowed some assets, even if only 1 wei.
```solidity
    function transferTokens(address spender, address src, address dst, uint tokens) internal returns (uint) {
        /* Fail if transfer not allowed */
        uint allowed = comptroller.transferAllowed(address(this), src, dst, tokens);
        if (allowed != 0) {
            revert TransferComptrollerRejection(allowed);
        }
        // ...
    }

    function transfer(address dst, uint256 amount) override external nonReentrant returns (bool) {
        return transferTokens(msg.sender, msg.sender, dst, amount) == NO_ERROR;
    }

    function transferFrom(address src, address dst, uint256 amount) override external nonReentrant returns (bool) {
        return transferTokens(msg.sender, src, dst, amount) == NO_ERROR;
    }
```

- USDT pause mode
```solidity
    function transfer(address _to, uint _value) public whenNotPaused {}

    function transferFrom(address _from, address _to, uint _value) public whenNotPaused {}
```

##### Reference:

AAVE:
https://github.com/aave/protocol-v2/blob/master/contracts/protocol/tokenization/base/DebtTokenBase.sol

Compound:
https://github.com/compound-finance/compound-protocol/blob/master/contracts/CToken.sol

USDT implementation:
https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code


All `transferERC721()` and `transferERC20()` will have no effects, just consuming transferEOAs' gas fund. When the token transfer guaranteed to fail, the for loop will consume large amount of gas, but no token would be transferred to the vault.
```solidity
// contracts\contracts\Transfer.sol

    function batchTransferERC721(ERC721Details[] memory _details) public returns (bool) {
        require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller.");
        for (uint256 i=0; i<_details.length; i++ ) {
            // If statement adds a bit more gas cost, but allows us to continue the loop even if a
            // token is not in a user's wallet anymore, instead of reverting the whole batch
            try this.transferERC721{gas:400e3}(_details[i].ownerAddress, _details[i].erc721Address, _details[i].erc721Id, _details[i].erc721Fee) {}
            catch {
                emit failedERC721Transfer(_details[i].ownerAddress, _details[i].erc721Address, _details[i].erc721Id);
            }
        }
        return true;
    }
    
    function batchTransferERC20(ERC20Details[] memory _details) public returns (bool) {
        require(_transferEOAs[msg.sender] == true, "Caller must be an approved caller.");
        for (uint256 i=0; i<_details.length; i++ ) {
            try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {}
            catch {
                emit failedERC20Transfer(_details[i].ownerAddress, _details[i].erc20Address);
            }
        }
        return true;
    }
```


## Tool used

Manual Review

## Recommendation

- Use a whitelist for token supported.
- Check the pause status of the assets (need to be specific to each token).
- Consider add some upfront fee, currently the fee is only collected when withdraw fund. But upfront fee can only prevent the griefing attack, not the miners.




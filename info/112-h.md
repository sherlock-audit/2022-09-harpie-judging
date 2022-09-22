__141345__
# Transfer inputs need validation to prevent stealing gas

## Summary

Token transfer parameters are not checked, if ERC20 transfer amount is 0, or ERC721 id belongs to another address, the original transfer is meaningless anyway, the transferEOAs will lose the high gas for front run. No token will be transferred but gas will be consumed. Eventually steal the transferEOAs' fund by unbounded gas cost. 

The miners have the incentive to perform this kind of attack, since they will directly benefit from the high gas fee. This can also be used to grief the system, because the cost of the attackers is low, just the lowest gas fee for one single transfer.


## Vulnerability Detail

Some users may want to trick the transferEOAs to frontrun with high gas fee, but at the end transfer no tokens. The followings transfers will have no effect or guaranteed reverts:
- 0 amount of ERC20 transfer
- random ERC721 id 
- flashloan pay back

0 amount transfer has no effect or will revert for some tokens. Random ERC721 id transfer will also revert. However, the checks and revert happened after the transaction sent to the mempool, the front run from the transferEOAs will still be triggered. As for the flashloan pay back, although some main protocols will be registered with harpie, there are still lots of other protocols provide this service probably not in the trusted network.

The miners benefit directly from high gas. To perform the attack, the aim is to get the gas fee as high as possible, batch transfer would be called, and the input array would be extremely long. 

The attacker only need to send one such transfer to untrusted address with the lowest gas fee, the `Transfer` contract will be triggered. Hence the cost for this attack is very low.



## Impact

The `Transfer` contract frontrun mechanism will be abused, transferEOAs will continuously loss fund due to high gas. Miners benefit from the high gas fee have the motivation for this attack. Griefing attack could also happen to influence the contract normal function. During chain congestion, the gas price will be much higher than normal times, making the loss even bigger.


## Code Snippet

All `transferERC721()` and `transferERC20()` will have no effects, just consuming transferEOAs' gas fund. Even no token transferred at the end, the for loop will consume large amount of gas, but no token would be transferred.
```solidity
contracts\contracts\Transfer.sol
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

- for ERC20, check transfer amount
make sure the transfer amount is > 0

- for ERC20, check balance before current block
make sure the balance is not from flashloan

- for ERC721, check the token ownership
```solidity
// contracts\contracts\Transfer.sol

    function transferERC721(address _ownerAddress, address _erc721Address, uint256 _erc721Id, uint128 _fee) public returns (bool) {
        // ...
        require(ownerOf(_erc721Id) == _ownerAddress);
```

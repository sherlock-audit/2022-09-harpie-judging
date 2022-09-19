__141345__
# Assets need validation and upfront fee might prevent abusing the system 

## Summary

Some asset are not worthy to protect, there are some cases the system will be abused, such as low balance ERC20, some NFT with expiry date in the near future. But currently some user may take the protection service as a free option, because the fee is only collected when withdraw fund, users may want the transferEOAs to secure the assets anyway, and defer the decision whether to pay to withdraw. However, if the protection is triggered, the transferEOAs will lose the high gas fee for front run, every time this is fund loss for them.

Charging upfront fee can help prevent abusing the system. If the users did really paid something at the beginning, at least it means they treat it seriously, other than take the transferEOAs and protection for free. A sanity check for the value of the assets will also help. 

## Vulnerability Detail

- Some asset are not worthy to protect
because the cost to front run might be even bigger than the assets. Such as low balance or low value ERC20.
- The value of the assets might change
if the price of asset being transferred to the vault collapse, the users are not willing to pay to withdraw.
- Some asset will expire
Such as ENS domain, which has an expiry time. Even being saved and transferred to the vault, the owner might not want to pay the fee to withdraw as the expiry approaching.

Another example is Putty protocol, which creates NFT for options, the NFT could be worthless or expiring soon. In this case, the NFT has both a time value with expiry and the value may vary significantly.

As NFTs has a wide applications, there are many possibilities that the value of the NFT will decay very fast. Then pay a fee to withdraw will be meaningless.

Since the payment for the protection is incurred only at withdraw, this rule effectively give the users a free option. They can wait to see whether it benefits them to pay the fee to withdraw. Maybe the ERC20 price goes up later, maybe the NFT gains value, then it makes sense to withdraw. But everything is not sure, there will be cases users have no motivation to pay the fee, leaving the loss to the transferEOAs.

## Impact

The protection system might be abused, some users could treat the protection like a free option. In some cases, the transferEOAs have to pay the high gas fee for front running, but later no payment for withdraw, causing fund loss and resources waste.


## Code Snippet

Payment is incurred when withdraw.
```solidity
// contracts\contracts\Vault.sol

    function withdrawERC20(address _originalAddress, address _erc20Address) payable external {}

    function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {}
```


## Tool used

Manual Review

## Recommendation

- Consider add some upfront fee, instead of charge only in withdraw.

- for ERC20, check address balance and value
maybe set a minimum amount, since it does not make sense to spent high gas to protect 1 wei transfer, this amount can be the accumulated amount within a time window.

```solidity
// contracts\contracts\Transfer.sol

    function transferERC20(address _ownerAddress, address _erc20Address, uint128 _fee) public returns (bool) {
        // ...
        require(balance > MIN_AMOUNT);
```
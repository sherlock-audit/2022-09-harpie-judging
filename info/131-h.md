HonorLt
# Risk of operating without any collateral

## Summary
Users are not required to deposit any upfront collateral to cover the fees.

## Vulnerability Detail
As far as I understand the current implementation does not require users to post any collateral beforehand. There are many risks with such an approach unless users are KYCed. What if the user never withdraws, e.g. the protocol is hacked and ERC20s lose value, or NFT becomes valueless? Or it becomes unprofitable to withdraw, e.g. when the value of tokens < fee + gas costs. This will result in a loss for both the user and the Harpie. Basically, there should be at least a minimal approach to enforcing that users are solvent to reclaim the assets.

## Tool used

Manual Review

## Recommendation
In my opinion, a safer option would be if users need upfront on-chain collateral for the fees, e.g. deposit 0.1 ETH or something like that. This collateral can even earn yield in blue chips DeFi (e.g. Aave).

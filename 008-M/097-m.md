ak1
# fee can be set to any arbitrary value by transferrer - lead to more centralized system.

## Summary
Transfer.sol : fee can be set to any arbitrary value by transferrer . This could lead to more centralized system.

## Vulnerability Detail
During the transfer of ERC20 and NFTs, the fee that is added can be any value. There are possibilities that the transferrer can set any higher value of fee.

## Impact
Medium/High

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Transfer.sol#L57

https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Transfer.sol#L88

## Tool used

Manual Review, VS code

## Recommendation
Add a function for fee calculation.
Call this function whenever funds are transferred. 
The fee calculation could be based on known percentage value. (%_fee * asset_value)
The percentage can be set by the admin and it  should be public so that anyone will aware of this.
Add function for fee setting.


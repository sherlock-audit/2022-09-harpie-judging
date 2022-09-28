magu
# abi.encodeCall will not work

## Summary

https://github.com/ethereum/solidity/issues/12435
Prior to pragma solidity^0.8.12, the abi.encodeCall function was reported to work only with function pointers and would not work in Transfer.sol.

## Vulnerability Detail

It is used in the transferERC20 and transferERC721 functions, so tokens may be moved in an unintended way.

## Impact
Transfer.sol cannot be deployed as it is because it probably returns an error at compile time.

## Code Snippet

https://github.com/sherlock-audit/2022-09-harpie-magugumagu/blob/d3f596b143470c056917c28beb3084a5969b3beb/contracts/contracts/Transfer.sol#L61

https://github.com/sherlock-audit/2022-09-harpie-magugumagu/blob/d3f596b143470c056917c28beb3084a5969b3beb/contracts/contracts/Transfer.sol#L65

https://github.com/sherlock-audit/2022-09-harpie-magugumagu/blob/d3f596b143470c056917c28beb3084a5969b3beb/contracts/contracts/Transfer.sol#L99


## Tool used

Manual Review

## Recommendation
Please update to pragma solidity~0.8.13 or later.
https://twitter.com/solidity_lang/status/1504162969114095620
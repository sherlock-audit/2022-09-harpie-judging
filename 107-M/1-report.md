ladboy233
# Message call with hardcoded gas amount when doing batch ERC20 token transfer ERC721 token transfer

## Summary

when doing batch ERC20 token transfer and ERC721 token transfer in Transfer.sol, the gas is hardcoded

```
     try this.transferERC721{gas:400e3}(_details[i].ownerAddress, _details[i].erc721Address, _details[i].erc721Id, _details[i].erc721Fee) {}
```

```
     try this.transferERC20{gas:400e3}(_details[i].ownerAddress, _details[i].erc20Address, _details[i].erc20Fee) {}
```

the hardcoded gas amount can break and the transfer can fail silently.

because the transfer is very time sensitive, either harpie lands the transaction to protect the fund or the hacker takes the fund.

## Vulnerability Detail

this is a known vulnerability 

https://swcregistry.io/docs/SWC-134

the hardcoded gas amount can break. for example, the underlying ERC20 may charge fee on transfer (do a swap before transfer)

However, the gas cost of EVM instructions may change significantly during hard forks which may break already deployed contract systems that make fixed assumptions about gas costs. For example. [EIP 1884](https://eips.ethereum.org/EIPS/eip-1884) broke several existing smart contracts due to a cost increase of the SLOAD instruction.

## Impact

the transaction may run out of gas and fail silently.

## Tool used

Manual inspection 

Foundry

## Recommendation

We recommend the project not hardcode the gas to avoid the issue.


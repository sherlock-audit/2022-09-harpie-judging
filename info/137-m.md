HonorLt
# On/off-chain coordination issues

## Summary
There are many potential issues regarding on/off-chain coordination and the trustable list of addresses. 
Sorry if this is a bit chaotic but because Watsons are discouraged to communicate with the team directly and the documentation does not cover all the cases, there are left some uncertainties about how the protocol is supposed to work.

## Vulnerability Detail
Users will have to initially approve for max in case of the token balance increases later. This needs to be done for every token one by one that needs to be protected. Consider adding approvals by signatures so users won't have to make on-chain txs for every token.

A trustable list is stored off-chain. A breach in web2 infrastructure can harm users. But keeping an on-chain list might also be ineffective.

The documentation mentions this:
**Since these transactions are self-signed, and our users are unlikely to use mempool obfuscators for their daily transactions,
we find that mempool obfuscation is not a concern for our use case.**
However, **Private Key Theft** is mentioned in the list of the types of attacks that Harpie can prevent. Thus leaving it unclear how mempool transactions are protected.

Another question I had is when the user is interacting with the Uniswap-like router. I assume the router will be on the list of trustable addresses. First, the user approves the router. And then the router is called with a parameter of a fake pool, how will Harpie prevent this?

## Tool used

Manual Review

## Recommendation
Re-think which components should be stored on-chain or off-chain and how to secure them.

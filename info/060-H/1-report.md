daejunpark
# Transfer.transferERC721() missing contract-existence-check may lead to NTFs stolen from vault

## Summary

Missing the contract-existence-check for low-level calls may lead to user NTFs being stolen from the vault.

## Vulnerability Detail

`Transfer.transferERC721()` makes a low-level call to `_erc721Address` _without_ checking the existence of the target address.  If the target address `_erc721Address` does NOT exist, the low-level call will silently return `true` with doing nothing (as explained in the [solidity docs](https://docs.soliditylang.org/en/v0.8.17/units-and-global-variables.html?highlight=non-existing#members-of-address-types)). If this happens, the vault will have an incorrect `_erc721WithdrawalAllowances` mapping entry for a non-existing NTF, which may be exploited to steal other users' NTFs. (See the exploit scenario below.)

https://github.com/sherlock-audit/2022-09-harpie-daejunpark/blob/9c44130f650b958483caade7da5aebd7ef2b116f/contracts/contracts/Transfer.sol#L60-L62

## Impact

My understanding of the intended security model is that, as long as the `serverSigner` is _not_ compromised,  user assets in the vault should be safe, even if the `transferEOA`s are compromised.  Below is a scenario that breaks the security model.

Exploit scenario: (A proof-of-concept exploit test can be found at: https://github.com/sherlock-audit/2022-09-harpie-daejunpark/blob/master/test/Exploit.t.sol )

0. Consider a permissionless NTF platform where artists can freely create their NTFs using the platform's factory contract, where the factory uses CREATE2 to _deterministically_ create new NTF addresses for good reasons.
1. Now, suppose artists plan to create their new NTF soon, and the address of the soon-to-be-created NTF is already known.
2. Knowing their plan, Bob somehow compromises or deceives a `transferEOA` so that `transferERC721()` is executed with `_ownerAddress` being Bob, `_erc721Address` being the soon-to-be-created NTF, and a certain `_erc721Id` X.
3. Note that, since `address(_erc721Address).call(...)` will immediately return because `_erc721Address` doesn't exist yet, `transferERC721()` will log an invalid transfer information in the vault without actually transferring the NTF.
4. Then, later, the new NTF contract is created into the known address, and minting starts.  Suppose Alice mints a NTF of id X, and registers for the Harpie protection service, as her current environment is unsafe and vulnerable for various phishing campaigns.
5. Then, Bob phishes Alice to transfer or approve her NTF, and Harpie detects and protects it by transferring her NTF to the vault.
6. Now, Bob calls `withdrawERC721()` and takes Alice's NTF.

Note that Bob may run step 2 with multiple ids, so that they can run the phishing campaigns against more users in step 5. Note that the attack cost is only the gas fee (in addition to their already existing phishing cost).

Besides the adversaries' profit, the refutation cost for Harpie will be very high if this exploit ever happens. Since users will anyway lose their assets when they are phished even if they registered for the Harpie service, they will complain that Harpie failed to deliver the promised protection against phishing.

## Code Snippet

See above.

## Tool used

Manual Review

## Recommendation

Add a contract-existence check for the low-level calls throughout the contract.

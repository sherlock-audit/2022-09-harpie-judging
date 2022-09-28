IllIllI
# Harpie does not protect against flash bots or malicious miners

## Summary
The "comprehensive" known risks page https://harpie.gitbook.io/welcome-to-the-harpie-docs/tech-and-security/disclosures-and-risks does not mention the fact that Harpie is not flash-bot resistant

## Vulnerability Detail
If an attacker uses a private mempool or is a miner themselves, it's not possible for Harpie to notice the transaction in time to protect the user, and user's funds will be lost

This page mentions flash bots in the context of the user using them himself/herself, but not in the context of an attacker doing it, and is not a listed risk https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/whitepaper#frequently-asked-questions

Attackers already know to use flash bots in order to avoid MEV, so doing it for normal transfers will be nothing new to them

## Impact
Users will be unprotected when they think they're protected

## Code Snippet
N/A

## Tool used

Manual Review

## Recommendation
This risk cannot be protected against, and should be explicitly mentioned. 
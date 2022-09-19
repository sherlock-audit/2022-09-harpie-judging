ak1
# changeRecipientAddress can be removed

## Summary
The function `changeRecipientAddress` can be removed. User can be allowed to update the recipient address as per their wish. 

## Vulnerability Detail
The function does not have purpose other than consuming some amount of gas.
I will add in loss of fund. Of course the value is less.

## Impact
Medium

## Code Snippet
https://github.com/sherlock-audit/2022-09-harpie-aktech297/blob/0d4cb554e1179a8cb25131b400178f69de23f080/contracts/contracts/Vault.sol#L62-L73


## Tool used
Manual Review,  VS code

## Recommendation
Allow the user to call the `function setupRecipientAddress` whenever they wish. Update that function as given below.
    `function setupRecipientAddress(address _recipient) external {
        require(_recipient != address(0), "Can not set zero addees");
       _recipientAddress[msg.sender] = _recipient;
    }`   
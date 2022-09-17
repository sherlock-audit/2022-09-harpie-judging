Bnke0x0
# User's may accidentally overpay in `require()` / `renew()` and the excess will be paid to the vault creator

## Summary
User's may accidentally overpay in `require()` / `renew()` and the excess will be paid to the vault creator
## Vulnerability Detail

## Impact
User's may accidentally overpay in `require()` / `renew()` and the excess will be paid to the vault creator

## Code Snippet

1. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Vault.sol#L122

            'require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");'

2. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Vault.sol#L133

             'require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");'    

3. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Vault.sol#L143

              'require(_erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee >= _reduceBy, "You cannot reduce more than the current fee.");'

4. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Vault.sol#L150

               'require(_erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee >= _reduceBy, "You cannot reduce more than the current fee.");'

5. File : https://github.com/sherlock-audit/2022-09-harpie-Bnke0x0/blob/master/contracts/contracts/Vault.sol#L158

                'require(address(this).balance >= _amount, "Cannot withdraw more than the amount in the contract.");'        

## Tool used

Manual Review

## Recommendation

Consider changing `>=`  to  `==`

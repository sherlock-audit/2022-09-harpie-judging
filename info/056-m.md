0xSmartContract
# Cross claim conflict in constructor arguments

## Summary
`Transaction.sol`  contract requests ```Vault.sol``` contract address in constructor and ```Transaction.sol``` contract address in ```Vault.sol``` contract. The deploy process is locked because the other contract cannot be deployed without knowing the previous address.

## Code Snippet

[Transfer.sol#L43-L46](https://github.com/sherlock-audit/2022-09-harpie-0xSmartContract/blob/master/contracts/contracts/Transfer.sol#L43-L46)
[Vault.sol#L48-L52](https://github.com/sherlock-audit/2022-09-harpie-0xSmartContract/blob/master/contracts/contracts/Vault.sol#L48-L52)


```js
Vault.sol: 

   // Need _transferer address (Transfer.sol contract address)
    constructor(address _transferer, address _serverSigner, address payable _feeController) {
        transferer = _transferer;
        serverSigner = _serverSigner;
        feeController = _feeController;
    }


Transfer.sol: 

   // Need _vaultAddress  address (Vault.sol contract address)
    constructor(address _vaultAddress, address _transferEOASetter) {
        vaultAddress = _vaultAddress;
        transferEOASetter = _transferEOASetter;
    } 
```

## Vulnerability Detail

We see how this issue is handled by the project in the deploy.js file:

[deploy.js#L14-L21](https://github.com/sherlock-audit/2022-09-harpie-0xSmartContract/blob/master/contracts/scripts/deploy.js#L14-L21)

```js
async function main() {
    // Other codes.......

    const transferAddress = getContractAddress({
      from: deployer.address,
      nonce: txCount
    })
    const vaultAddress = getContractAddress({
      from: deployer.address,
      nonce: txCount + 1
    })

    let transferContract = await Transfer.deploy(vaultAddress, "0xeE5437fBc370aBf64BB8E855824B01485C497F49");
    let vaultContract = await Vault.deploy(transferAddress, "0x54CBE75825d6f937004e37dc863258C4f4AdDc58", "0x9ae1fE63a9150608DfBb644fdf144595244619C3");

```

Due to the ```async function main```, the other contract address will not be created before a contract address is formed, it is observed that this is solved with the formula with ```Nonce+1```. However, it should be noted that if the transaction fee may be reverted for reasons such as insufficient fee, it is also not a best practice.


## Tool used

Manual Review

## Recommendation
The addressing architecture should be changed and adding addresses should be done without addressing conflicts, as in other projects (for example proxy contracts)
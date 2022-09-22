0xSmartContract
# Transactions can be made by paying 0 fee

## Summary
The target of the project is to get a fee from each transaction, in this fee, the function in the Transfer contract is entered during the first transfer transaction.

However, fee can be logged as 0 during a transaction and has no control over it. In addition, with the latest withdrawal, there is no control during the exit of token or NFT from the vault.

It is stated in the documents that there is a 7% transaction fee.
https://harpie.gitbook.io/welcome-to-the-harpie-docs/about/pricing

## Code Snippet

1 - First of all, Transfer contract's ```transferERC721```function is run with 0 fee (```uint128 _fee``` : ```0```)
  - There is no control about the fee  at this stage 
  -  Only on line 65, 0 is transmitted to the Vault contract. (```Vault.logIncomingERC721```)

```js

2022-09-harpie-contracts/contracts/Transfer.sol:

  57:     function transferERC721(address _ownerAddress, address _erc721Address, uint256 _erc721Id, uint128 _fee) public returns (bool) {
  58:         require(_transferEOAs[msg.sender] == true || msg.sender == address(this), "Caller must be an approved caller.");
  59:         require(_erc721Address != address(this));
  60:         (bool transferSuccess, bytes memory transferResult) = address(_erc721Address).call(
  61:             abi.encodeCall(IERC721(_erc721Address).transferFrom, (_ownerAddress, vaultAddress, _erc721Id))
  62:         );
  63:         require(transferSuccess, string (transferResult));
  64:         (bool loggingSuccess, bytes memory loggingResult) = address(vaultAddress).call(
  65:             abi.encodeCall(Vault.logIncomingERC721, (_ownerAddress, _erc721Address, _erc721Id, _fee))
  66:         );
  67:         require(loggingSuccess, string (loggingResult));
  68:         emit successfulERC721Transfer(_ownerAddress, _erc721Address, _erc721Id);
  69:         return transferSuccess;
  70:     }

```

2 - The value 0 passed by the ```transferERC721``` function is add to mapping  in the ```logIncomingERC721``` function below (```uint128 _fee``` : 0)

  - There is no control about the fee  at this stage 
  -  Only on line 91, ```0``` fee add to mapping

```js
2022-09-harpie-contracts/contracts/Vault.sol:
  88  
  89:     function logIncomingERC721(address _originalAddress, address _erc721Address, uint256 _id, uint128 _fee) external {
  90:         require(msg.sender == transferer, "Only the transferer contract can log funds.");
  91:         _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee += _fee;
  92:         _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].isStored = true;
  93:     }
  94  
```

3 - At this stage, we can confirm that it is withdrawable  (```uint128 _fee``` : ```0```)
```js
2022-09-harpie-contracts/contracts/Vault.sol:
  100  
  101:     function canWithdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) public view returns (bool) {
  102:         return _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].isStored;
  103:     }
  104  
````


4 - User withdraws her NFT from the platform with withdrawERC721 (```uint128 _fee``` : ```0```)

  -  Only on line 131 It is checked whether ```msg.value``` is equal to or greater than ```fee``` .Note that the fee mapping value is 0, we can skip this part with ```0``` in the ```msg.value```
 

```js

2022-09-harpie-contracts/contracts/Vault.sol:

  129:     function withdrawERC721(address _originalAddress, address _erc721Address, uint256 _id) payable external {
  130:         require(_recipientAddress[_originalAddress] == msg.sender, "Function caller is not an authorized recipientAddress.");
  131:         require(_erc721Address != address(this), "The vault is not a token address");
  132:         require(canWithdrawERC721(_originalAddress, _erc721Address, _id), "Insufficient withdrawal allowance.");
  133:         require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
  134: 
  135:         _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].isStored = false;
  136:         _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee = 0;
  137:         IERC721(_erc721Address).transferFrom(address(this), msg.sender, _id);
  138:     }

```

5- As a result, the fee specified in the documents can be passed with 0 fee entry.


## Proof Of Concept
The following is the proof of concept that in the project's own Test contract, the transaction is carried out with a 0 fee 

Lines with 0 fee [ Line 7   / Line 76 / Line 112]

```js
test/Permissions-Tests.js:
    1: // We import Chai to use its asserting functions here.
    2: const { expect } = require("chai");
    3: const { BigNumber } = require("ethers");
    4: const { waffle, ethers } = require("hardhat");
    5: const { getContractAddress } = require("@ethersproject/address");
    6: const NULL_ADDRESS = "0x0000000000000000000000000000000000000000";
    7: const EXACT_FEE = { value: 0 };
    8: const OVER_FEE = { value: 101 };
    9: const UNDER_FEE = { value: 99 };
   10: const ZERO = { value: 0 };
   11: 
   12: describe("Transfer contract", function () {
   13: 
   14:     let NFT1;
   15:     let NFT2;
   16:     let Transfer;
   17:     let Vault;
   18:     let nftContract1;
   19:     let transferContract;
   20:     let vaultContract;
   21:     let user;
   22:     let addr1;
   23:     let addr2;
   24:     let recipientAddr;
   25:     let addrs;
   26: 
   27:     // These tests all run synchronously: tests will not repeat themselves
   28:     before(async function () {
   29:         NFT1 = await ethers.getContractFactory("NFT");
   30:         NFT2 = await ethers.getContractFactory("NFT");
   31:         Token1 = await ethers.getContractFactory("Fungible");
   32:         Token2 = await ethers.getContractFactory("Fungible");
   33:         Transfer = await ethers.getContractFactory("Transfer");
   34:         Vault = await ethers.getContractFactory("Vault");
   35:         [user, serverSigner, transferSignerSetter, transferSigner, feeController, feeController2, addr1, addr2, recipientAddr, ...addrs] = await ethers.getSigners();
   36: 
   37:         // Determine upcoming contract addresses
   38:         const txCount = await user.getTransactionCount();
   39:         const transferAddress = getContractAddress({
   40:             from: user.address,
   41:             nonce: txCount
   42:         })
   43: 
   44:         const vaultAddress = getContractAddress({
   45:             from: user.address,
   46:             nonce: txCount + 1
   47:         })
   48: 
   49:         transferContract = await Transfer.deploy(vaultAddress, transferSignerSetter.address);
   50:         vaultContract = await Vault.deploy(transferAddress, serverSigner.address, feeController.address);
   51:         nftContract1 = await NFT1.deploy();
   52: 
   53: 
   54:         await nftContract1.mint(user.address);
   55:  
   56:      
   57:     });
   58: 
   59:     describe("Deployment", async () => {
   60:         it("User address should approve Transfer address for NFTs", async () => {
   61:             await nftContract1.setApprovalForAll(transferContract.address, true);
   62:             expect(await nftContract1.isApprovedForAll(user.address, transferContract.address)).to.equal(true);
   63:         })
   64: 
   65:     })
   66: 
   67:     describe("Transfer: transferEOA permissions", async () => {
   68: 
   69:         it("Only transferEOASetter can call setTransferEOA", async () => {
   70:             await expect(transferContract.connect(serverSigner).setTransferEOA(serverSigner, true)).to.be.reverted;
   71:             await transferContract.connect(transferSignerSetter).setTransferEOA(transferSigner.address, true);
   72:             await transferContract.connect(transferSignerSetter).setTransferEOA(addr1.address, true);
   73:         })
   74: 
   75:         it("Should revert transactions when a wallet is no longer in the transferEOAs map", async () => {
   76:             await transferContract.connect(addr1).transferERC721(user.address, nftContract1.address, 1, 0);
   77:             await transferContract.connect(transferSignerSetter).setTransferEOA(addr1.address, false);
   78:            
   79:         })
   80:     })
   81: 
   82:     describe("Vault: User registration", async () => {
   83: 
   84:         it("Should successfully allow a user to set a recipientAddress", async () => {
   85:             await vaultContract.setupRecipientAddress(addr1.address);
   86:             expect(await vaultContract.viewRecipientAddress(user.address)).to.equal(addr1.address);
   87:         })
   88: 
   89:         const getExp = async (offset) => {
   90:             const provider = waffle.provider;
   91:             const blockNum = await provider.getBlockNumber();
   92:             const block = await provider.getBlock(blockNum);
   93:             const exp = block.timestamp + offset;
   94:             return exp;
   95:         }
   96: 
   97:         it("Should successfully allow a user to change a recipientAddress", async () => {
   98:             const exp = await getExp(15 * 60);
   99:             const messageHash = ethers.utils.solidityKeccak256(['address', 'address', 'uint256', 'address'], [user.address, recipientAddr.address, exp, vaultContract.address]);
  100:             const messageHashBinary = ethers.utils.arrayify(messageHash);
  101:             const signature = await serverSigner.signMessage(messageHashBinary);
  102:             await vaultContract.changeRecipientAddress(signature, recipientAddr.address, exp)
  103: 
  104:             expect(await vaultContract.viewRecipientAddress(user.address)).to.equal(recipientAddr.address);
  105:         })
  106:     })
  107: 
  108:     describe("Vault: Withdrawing ERC721s", async () => {
  109: 
  110:         it("Should allow a user to withdraw a single NFT after setting recipientAddress", async () => {
  111:             expect(await nftContract1.ownerOf(1)).to.equal(vaultContract.address);
  112:             await vaultContract.connect(recipientAddr).withdrawERC721(user.address, nftContract1.address, 1, EXACT_FEE);
  113:             expect(await nftContract1.ownerOf(1)).to.equal(recipientAddr.address);
  114:         })
  115:     })
  116: });
```

## Tool used

Manual Review

## Recommendation
A require should be added to control the amount of fee
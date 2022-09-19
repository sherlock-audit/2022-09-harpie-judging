tofunmi
# The Vault contract doesn't Implement the EIP712 IERC721Receiver function to successfully receive NFTs

## Summary
- The vault is not an EOA and deosn't implement the **_IERC721Receiver.onERC721Receive_** token implementation from **_openzeppelin_** and the eip specification **_[eip712](#)_**
  - This means transfering a EIP712 NFT to the vault from the transferer through transferFrom would always contract would always fail.
## Details
- From the openzzeppelin's docs iteself https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver, if an NFT receiver is a contract and not an EOA , it must implement the [IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-), which is called upon a safe transfer. 
## Tool used
vscode ,hardhat  and Manual Review

## Recommendation
- Implement or inherit the IERC721Receiver.onERC721Received](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver-onERC721Received-address-address-uint256-bytes-) , to safely receive NFTs 
``` solidity
import "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
contract Vault is  IERC721Receiver {
     .....
      function onERC721Received(address, uint256, bytes) public returns(bytes4) {
        return  bytes4(keccak256("onERC721Received(address,uint256,bytes)"));
    }
}
```

Tomo
# Too much paid ETH is lost

## 💥 Impact
If users too much paid ETH in the `withdrawERC20` and `withdrawERC721`functions, this ETH is lost.

## 🕵️‍♂️ Vulnerability Detail
### _Exploit Scenario_
1. Alice executes the `withdrawERC20` function to get some ERC20 Tokens as follows. To do this, Alice needs to pay 1 wei as a fee
2. However, Alice mistakenly set `msg.value` to 1 ETH.
3. After executing this function, Alice noticed too much paid ETH.
4. Finally, Alice can’t get too much paid ETH.

## 📝 Code Snippet
```solidity
function withdrawERC20(/* ... */) payable external {
    /* ... */
    require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
    /* ... */
}

function withdrawERC721() payable external {
    /* ... */
    require(msg.value >= _erc721WithdrawalAllowances[_originalAddress][_erc721Address][_id].fee, "Insufficient payment.");
    /* ... */
}
```

## 🚜 Tools Used

Manual

## ✅ Recommendation

You should choose one of these options

1. Ensure the fee equal to `msg.value` before transferring ERC20 tokens or NFT.

```solidity
require(msg.value == _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, 
"Insufficient payment.");
```

2. If `msg.value` is too much paid, refund to `msg.sender`

```solidity
function withdrawERC20(/* ... */) external payable {
	 /* ... */
	 uint beforeBalance = address(this).balance;   
	 require(msg.value >= _erc20WithdrawalAllowances[_originalAddress][_erc20Address].fee, "Insufficient payment.");
	 /* ... */
	 IERC20(_erc20Address).safeTransfer(msg.sender, amount);
	 // If msg.value is too much, this value will be more than 0
	 uint gap = address(this).balance - beforeBalance;
	 if(afterBalance - beforeBalance > 0) {
     (bool success, bytes memory data) = msg.sender.call{value: gap }("");
     require(success, "Failed to send Ether");
 }
}
```

## 👬 Similar Issue

https://github.com/code-423n4/2022-03-lifinance-findings/issues/185

https://github.com/code-423n4/2022-06-infinity-findings/issues/346
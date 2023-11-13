# Risk
Low [1]

# Title
The input argument `_tokenid` is not checked if it is valid in `AuctionDemo.sol` contract

## Impact
In the functions: `participateToAuction()`, `returnHighestBid()`, `returnHighestBidder()`, `claimAuction()`, `cancelBid()`, `cancelAllBids()` and `returnBids()` in `AuctionDemo.sol` contract the input argument `_tokenid` is not checked if it is a valid token id.
Add a check in all functions in the contract `AuctionDemo.sol` that the `_tokenid` is valid.

## Links
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L57
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L65
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L104
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L124
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L134
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L147

# Risk
Low [2]

# Title
The input argument `_ethRequired` is not checked if it is equal to the required `ethRequired` variable in `requestRandomWords()` function in `RandomizerRNG.sol` contract

## Impact
In the `RandomizerRNG.sol` contract is defined a varible `ethRequired` that defines the required value of ether for the `arrngController.requestRandomWords` function. Also, this value is updated through the `updateRNGCost()` function.
The problem is that the `requestRandomWords()` function that makes external call to the `arrngController.requestRandomWords()` function, does not check if the value ot the input argument `_ethRequired` is equal to the value of `ethRequired` variable. If the `_ethRequired` is less than the actual required amount, it could lead to insufficient gas. The `requestRandomWords()` function might not have enough gas to complete its execution. This could cause the function to fail, potentially disrupting the operation of the contract.

```solidity
function requestRandomWords(uint256 tokenid, uint256 _ethRequired) public payable {
    require(msg.sender == gencore);
    uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
    tokenToRequest[tokenid] = requestId;
    requestToToken[requestId] = tokenid;
}
```
## Links
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L40-L46

## Recommendation
To mitigate these risks, it's crucial to ensure that the `_ethRequired` value is always sufficient to cover the gas costs of the `requestRandomWords` function. This could be achieved by adding a check in the function to compare `_ethRequired` with the `ethRequired` state variable.

# Risk
Low [3]

# Title
Division before multiplication in `MinterContract.sol` contract

## Impact

Division before multiplication is performed in `getPrice()` function in `MinterContract.sol` contract. In Solidity, the order of operations matters, especially when it comes to division and multiplication. This is because division and multiplication are not commutative operations, meaning the order in which they are performed can lead to different results. The performed division before multiplication can lead to incorrect price of collection due loss of precision. There are 3 instances of this problem:

`MinterContract.sol:536`
```return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L536

`MinterContract.sol:551`
```decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L551

`MinterContract.sol:554`
```price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);```

## Links
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L554

## Recommendation
Rewrite the `getPrice()` function in such a way that the division is performed at the end of the calculations.
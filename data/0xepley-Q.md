## 1): Add check to ensure that collection is already frozen
In the `freezeCollection` function of `NextGenCore.sol` there isn't any kind of check to ensure if the collection is already frozen or not, the only `require` statement that the function haves is to check if the collection exists which is as follow
```solidity
function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
        require(isCollectionCreated[_collectionID] == true, "No Col"); //@audit QA: what if collection is already
        collectionFreeze[_collectionID] = true;
    }
```
If the collection is already frozen there is no point in calling this function as the caller will loose gas by double calling it, so it is recommended to add a check to ensure that is collection is not freeze before calling this function so even if someone tries to call it second time he will see an error.


## 2): Wrong Natspec in `mint` function
In the `mint` function of `MinterContract.sol` in the following line
```solidity
        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
```
we can see that user must send either equal or more amount of ether and if he send less then that then the function will revert but the error message this function is displaying is very vague and can easily distract anyone, the only condition this line of code will revert is when the `msg.value` is less the `(getPrice(col) * _numberOfTokens` so make sure to display error message that is more relevant. 


## 3): Duplicate line in `burnToMint` function.
In `burnToMint` function of `MinterContract.sol` there are two lines that are exactly similar to each other with just 1 difference (variable name). The occurrence of two similar kind in the same function if useless and do not have any advantage instead it will only cost more gas price to the user so it is better to remove the extra line.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L263-L267

## 4): Do not use `uint256` in for loop

`uint256` costs more gas and occupy more space and using it in `for loop` is not necessary. Lets say there is a for loop that will run only to some number like 100, 1000 or maybe 10000 and assigning uint value to the `ith` index is vague. Make sure to adjust the `uint` value for better. Functions like `airDropTokens` `mint` are using `for loop` with uint256 which is not necessary

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L184

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L187

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L234

## 5  No check that `_numberOfTokens` and `_recipients`

In the function `airDropTokens` of `MinterContract.sol` admin is airdropping some tokens to the list of `_recipients` but the issue is that there is no check on the length of token array and the recipients array.
```solidity
 function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
    //    @audit no check that _numberOfTokens and _recipients must be same 
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
            }
        }
    }
```

Make sure to include a check that the length of both of these things must be equal to avoid unintended consequences.

## 6): Disperency between require/if statement

In the function `getPrice` there is an `if` statement which is as follow
```solidity
if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){
```
but if we see that on `2` different places in the same smart contract the above line is written as follow
```solidity
block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
```
see the difference between allowing execution at deadline. It is recommended to follow one model, either allow execution at deadline or don't allow at all

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L345
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L202

## 7): Last word is not accessible
In the function `getWord` of `RandomPool` there is an `if/else` block, these conditions are the mastermind to choose which word from the array will get select but the issue is that whatever we do the last word 100 word will always be skipped.


https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L28-L33

as you can see in the above code that even if we select `id` as 100 then in the `else` statement it will subtract 1 from the id so it will become `99` so it will choose 99th word not the last one.

## 8): `createCollection` can revert

`createCollection` is taking alot of string arguments as an input it is storing them as it is (in string format) now the issue is that long string can cause gas issue and can even revert causing loss of gas for the user. so the recommended step is to hash the string arguments before storing, it will reduce their size very much and will be favorable to store
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L130-L141
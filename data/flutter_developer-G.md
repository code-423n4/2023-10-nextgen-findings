
### Gas optimization 

|      |   issue  | instance |
|------|----------|----------|

| [G-01] | Core is the contract where the ERC721 tokens are minted and includes all the core functions of the ERC721 standard as well as additional setter & getter functions.|

| [G-02]| The Minter contract is used to mint an ERC721 token for a collection on the Core contract based on certain requirements that are set prior to the minting process.|


| [G-03] | The Admin contract is responsible for adding or removing global or function-based admins who are allowed to call certain functions in both the Core and Minter contracts.|


| [G-04] | 	The RandomizerNXT contract is responsible for generating a random hash for each token during the minting process using the NextGen's proposed approach.|



| [G-5] | The RandomizerVRF contract is responsible for generating a random hash for each token during the minting process using the Chainlink's VRF service.|



| [G-06] | The RandomizerRNG contract is responsible for generating a random hash for each token during the minting process using the ARRng.io service.|

| [G-07] | The randomPool smart contract is used by the RandomizerNXT contract, once it's called from the RandomizerNXT smart contract it returns a random word from the current word pool as well as a random number back to the RandomizerNXT smart contract which uses those values to generate a random hash.|


| [G-08] The auctionDemo smart contract holds the current auctions after the mintAndAuction functionality is called. Users can bid on a token and the highest bidder can claim the token after an auction finishes|


### Code Description 


### [G-01] Core is the contract where the ERC721 tokens are minted and includes all the core functions of the ERC721 standard as well as additional setter & getter functions.


```solidity
file: smart-contracts/NextGenCore.sol
```
    constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
        adminsContract = INextGenAdmins(_adminsContract);
        newCollectionIndex = newCollectionIndex + 1;
        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L108-L112
```

###  [G-02] The Minter contract is used to mint an ERC721 token for a collection on the Core contract based on certain requirements that are set prior to the minting process.


```solidity
file: main/smart-contracts/MinterContract.sol
```
    function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
        collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
        collectionPhases[_collectionID].rate = _rate;
        collectionPhases[_collectionID].timePeriod = _timePeriod;
        collectionPhases[_collectionID].salesOption = _salesOption;
        collectionPhases[_collectionID].delAddress = _delAddress;
        setMintingCosts[_collectionID] = true;
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157-L166
```



### [G-03]  The Admin contract is responsible for adding or removing global or function-based admins who are allowed to call certain functions in both the Core and Minter contracts.


```solidity
file: /main/smart-contracts/NextGenAdmins.sol
```
    function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
        for (uint256 i=0; i<_selector.length; i++) {
            functionAdmin[_address][_selector[i]] = _status;
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L50-L54
```





### [G-04]  	The RandomizerNXT contract is responsible for generating a random hash for each token during the minting process using the NextGen's proposed approach.|



```solidity
file: main/smart-contracts/RandomizerNXT.sol
```
        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L57
```




### [G-05] The RandomizerVRF contract is responsible for generating a random hash for each token during the minting process using the Chainlink's VRF service.


```solidity
file: main/smart-contracts/RandomizerVRF.sol
```
   require(msg.sender == gencore);
        tokenIdToCollection[_mintIndex] = _collectionID;
        requestRandomWords(_mintIndex);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L72-L74
```



### [G-06] The RandomizerRNG contract is responsible for generating a random hash for each token during the minting process using the ARRng.io service.

```solidity
file:main/smart-contracts/RandomizerRNG.sol 
```
        uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L42
```


### [G-07] The randomPool smart contract is used by the RandomizerNXT contract, once it's called from the RandomizerNXT smart contract it returns a random word from the current word pool as well as a random number back to the RandomizerNXT smart contract which uses those values to generate a random hash.

```solidity
file:main/smart-contracts/XRandoms.sol

```
uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
        return getWord(randomNum);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L41-L42
```


### [G-8] The auctionDemo smart contract holds the current auctions after the mintAndAuction functionality is called. Users can bid on a token and the highest bidder can claim the token after an auction finishes.


```solidity
file: main/smart-contracts/AuctionDemo.sol
```
   for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L69-L73
```


# ```freezeCollection``` should check if collection is not already frozen

In [```NextGenCore.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol)

```
    function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
        require(isCollectionCreated[_collectionID] == true, "No Col");
        collectionFreeze[_collectionID] = true;
    }
```

This would prevent unnecessary function execution.

# Use consistent naming conventions

[```auctionDemo```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L18) contract should be named ```AuctionDemo```

# Spacing of comments inconsistent

Example in RandomizerRNG.sol contract

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L52-L64

    // function that calculates the random hash and returns it to the gencore contract
    function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
        require(msg.sender == gencore);
        tokenIdToCollection[_mintIndex] = _collectionID;
        requestRandomWords(_mintIndex, ethRequired);
    }

    // function to update contracts

    function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
        adminsContract = INextGenAdmins(_newadminsContract);
    }


## Mitigation Steps

Keep consistent spacing between comments on their associated functions for better readability.

# Imports should be named for better readability

32 occurrences 

## Mitigation steps

Named the contracts / libraries that are actually used in the current contract. This will increase readability for users / auditors.

# Remove useless ```else``` statements

2 occurrences in [```AuctionDemo.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol)

```
    function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }
    }

```

And 

```
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
        uint256 highestBid = returnHighestBid(_tokenid);
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
        address highestBidder = returnHighestBidder(_tokenid);
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }

```

## Mitigation steps

Remove the ```else``` part of the ```if``` statement if it is not doing anything.

# Group variable definitions by type to improve readability

Grouping variables by type will make it easier for external readers to read / audit.

# Use better spacing between events and variable definitions.

1 occurrence in [```RandomizerRNG.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol)

```
    mapping(uint256 => uint256) public requestToToken;
    address gencore;
    INextGenCore public gencoreContract;
    INextGenAdmins private adminsContract;
    event Withdraw(address indexed _add, bool status, uint256 indexed funds);
    uint256 ethRequired;
    mapping(uint256 => uint256) public tokenToRequest;
    mapping(uint256 => uint256) public tokenIdToCollection;

```


# Don’t set new variable names for function parameters when those are already clear enough

Multiple occurrences throughout the code base.
Example in MinterContract.sol

```
        string memory tokData = _tokenData;
```

In the [```mint```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196-L254) function

``` 
function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        uint256 col = _collectionID;
        address mintingAddress;
        uint256 phase;
        string memory tokData = _tokenData;

```

# Functions need more comments for reader to understand inputs / outputs

Multiple occurrences throughout the code base. Very few function have thorough commenting with definition of inputs and outputs.

# ```newCollectionIndex``` should be set to ```1``` in the constructor.

In [```NextGenCore.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol), the constructor increments ```newCollectionIndex``` like below. This is useless as the ```newCollectionIndex``` is not initialized, meaning this statement will always set newCollectionIndex to 1.

```
    constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
        adminsContract = INextGenAdmins(_adminsContract);
        newCollectionIndex = newCollectionIndex + 1;
        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
    }

```

## Mitigation steps
Initialize the ```newCollectionIndex``` to 1, either in variable declaration directly or in the constructor.

# Typos

Multiple occurrences throughout the code base.

Example in [```MinterContract.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol):

```
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
```


# Inconsistent ordering of function parameters

Example in [```NextGenAdmins.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol)

The parameter ```_address``` could be in first position for both functions to increase readability.

```
    function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
        for (uint256 i=0; i<_selector.length; i++) {
            functionAdmin[_address][_selector[i]] = _status;
        }
    }

    // function to register a collection admin

    function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
        require(_collectionID > 0, "Collection Id must be larger than 0");
        collectionAdmin[_address][_collectionID] = _status;
    }
```

# Extensive use of hardcoded values

Multiple occurrences throughout the code base.

Examples in [```MinterContract.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol):

```
        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
        teamRoyalties1 = royalties * _teamperc1 / 100;
        teamRoyalties2 = royalties * _teamperc2 / 100;
```

```
            if (_delegator != 0x0000000000000000000000000000000000000000) {
                bool isAllowedToMint;
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
                if (isAllowedToMint == false) {
```

And [```NextGenCore.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol)

```
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
            collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
            collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
            collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
            collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
```

This is prone to errors especially when the same value is hardcoded several times.

# Use modifiers where applicable

Use modifiers instead of using several times the same check to improve efficiency and readability.

Multiple occurrences throughout the code base.

Example in [```NextGenCore.sol```](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol)

```
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
```

Is used several times and should be turned into a modifier.

## Mitigation steps
Define hardcoded values as constant or immutable values for better readability, and add comments to explain their meaning when naming is not clear enough for user to read.

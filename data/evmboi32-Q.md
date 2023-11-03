## Verify mint index in ```_mintProcessing```
```solidity
function _mintProcessing(
    uint256 _mintIndex,
    address _recipient,
    string memory _tokenData,
    uint256 _collectionID,
    uint256 _saltfun_o 
) internal {
+   require(tokenData[_mintIndex] == "", "Already used!");
    tokenData[_mintIndex] = _tokenData;
    collectionAdditionalData[_collectionID].randomizer.calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
    tokenIdsToCollectionIds[_mintIndex] = _collectionID;

    _safeMint(_recipient, _mintIndex);
}
```

## It is more elegant to use bytes for the salt instead of uint256
### NextGenCore.sol
```solidity
function airDropTokens(
    ...
    uint256 _saltfun_o,
    ...
) 
```

```solidity
function mint(
    ...
    uint256 _saltfun_o,
    ...
) 
```
```solidity
function burnToMint(
    ...
    uint256 _saltfun_o,
    ...
)
```
```solidity
function _mintProcessing(
    ...
    uint256 _saltfun_o 
)
```
### MinterContract.sol
```solidity
function airDropTokens(
    ...
    uint256[] memory _saltfun_o,
    ...
)
```
```solidity
function mint(
    ...
    uint256 _saltfun_o
)
```
```solidity
function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o)
```
```solidity
function mintAndAuction(
    ...
    uint256 _saltfun_o,
    ...
)
```

```solidity
function burnOrSwapExternalToMint(
    ...
    uint256 _saltfun_o
)
```

## Random index values used for changing values
```solidity
    function updateCollectionInfo(
        uint256 _collectionID,
        string memory _newCollectionName,
        string memory _newCollectionArtist,
        string memory _newCollectionDescription,
        string memory _newCollectionWebsite,
        string memory _newCollectionLicense,
        string memory _newCollectionBaseURI,
        string memory _newCollectionLibrary,
        uint256 _index,
        string[] memory _newCollectionScript
    ) public CollectionAdminRequired(_collectionID, this.updateCollectionInfo.selector) {
        require(
            (isCollectionCreated[_collectionID] == true) && (collectionFreeze[_cauollectionID] == false), "Not allowed"
        );
        if (_index == 1000) {
            collectionInfo[_collectionID].collectionName = _newCollectionName;
            collectionInfo[_collectionID].collectionArtist = _newCollectionArtist;
            collectionInfo[_collectionID].collectionDescription = _newCollectionDescription;
            collectionInfo[_collectionID].collectionWebsite = _newCollectionWebsite;
            collectionInfo[_collectionID].collectionLicense = _newCollectionLicense;
            collectionInfo[_collectionID].collectionLibrary = _newCollectionLibrary;
            collectionInfo[_collectionID].collectionScript = _newCollectionScript;
        } else if (_index == 999) {
            collectionInfo[_collectionID].collectionBaseURI = _newCollectionBaseURI;
        } else {
            collectionInfo[_collectionID].collectionScript[_index] = _newCollectionScript[0];
        }
    }
```
Consider adding a parameter that indicates which part of the config is being changed.

## Randomizers use an address and an instance of a gencore contract
### RandomizerRNG.sol
```solidity
constructor(address _gencore, address _adminsContract, address _arRNG) ArrngConsumer(_arRNG) {
    gencore = _gencore;
    gencoreContract = INextGenCore(_gencore);
    adminsContract = INextGenAdmins(_adminsContract);
}
```

### RandomizerVRF.sol
```solidity
constructor(uint64 subscriptionId, address vrfCoordinator, address _gencore, address _adminsContract)
    VRFConsumerBaseV2(vrfCoordinator)
{
    COORDINATOR = VRFCoordinatorV2Interface(vrfCoordinator);
    s_subscriptionId = subscriptionId;
    gencore = _gencore;
    gencoreContract = INextGenCore(_gencore);
    adminsContract = INextGenAdmins(_adminsContract);
}
```

### RandomizerNXT.sol
```solidity
constructor(address _randoms, address _admin, address _gencore) {
    randoms = IXRandoms(_randoms);
    adminsContract = INextGenAdmins(_admin);
    gencore = _gencore;
    gencoreContract = INextGenCore(_gencore);
}
```

Consider only using ```gencoreContract = INextGenCore(_gencore);``` and just using ```address(gencoreContract)``` whenever you need the address.

## Missing parameters check in ```setCollectionCosts```
```solidity
function setCollectionCosts(
    uint256 _collectionID,
    uint256 _collectionMintCost,
    uint256 _collectionEndMintCost,
    uint256 _rate,
    uint256 _timePeriod,
    uint8 _salesOption,
    address _delAddress
) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
    require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
+   require(_timePeriod > 0, "time period 0");
+   require(_salesOption >= 1 && _salesOption <=3, "_salesOption out of bounds.");
    
    collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
    collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
    collectionPhases[_collectionID].rate = _rate;
    collectionPhases[_collectionID].timePeriod = _timePeriod;
    collectionPhases[_collectionID].salesOption = _salesOption;
    collectionPhases[_collectionID].delAddress = _delAddress;
    setMintingCosts[_collectionID] = true;
}
```

## Missing checks in ```setCollectionPhases```
```solidity
function setCollectionPhases(
    uint256 _collectionID,
    uint256 _allowlistStartTime,
    uint256 _allowlistEndTime,
    uint256 _publicStartTime,
    uint256 _publicEndTime,
    bytes32 _merkleRoot
) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
    require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
    require(block.timestamp > _allowlistStartTime, "already started");
    require(_allowlistEndTime > _allowlistStartTime, "cannot end before start");
    require(_publicStartTime > _allowlistEndTime, "cannot start before end of AL");
    require(_publicEndTime > _publicStartTime, "cannot end before start");
    
    collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
    collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
    collectionPhases[_collectionID].merkleRoot = _merkleRoot;
    collectionPhases[_collectionID].publicStartTime = _publicStartTime;
    collectionPhases[_collectionID].publicEndTime = _publicEndTime;
}
```

## Incorrect tokenData in ```airDropTokens```
```solidity
function airDropTokens(
    address[] memory _recipients,
    string[] memory _tokenData,
    uint256[] memory _saltfun_o,
    uint256 _collectionID,
    uint256[] memory _numberOfTokens
) public FunctionAdminRequired(this.airDropTokens.selector) {
    require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
    uint256 collectionTokenMintIndex;
    for (uint256 y = 0; y < _recipients.length; y++) {
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID)
            + _numberOfTokens[y] - 1;
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
        for (uint256 i = 0; i < _numberOfTokens[y]; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
            gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
        }
    }
}
```

Docs say that ```@param _tokenData[] Refers to the additional token data that are stored on-chain for each airdropped token.```

The function is using ```_tokenData[y]``` which will make the ```tokenData``` the same for all tokens the recipients receive. It should be using ```_tokenData[i]``` (the index of the internal loop instead of the outer loop) to have different data for each token.

## Minting phases should use ENUM instead of magic numbers
While minting the magic numbers are used for phases

```solidity
function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
    ...
    uint256 phase;
    ...
    if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
        phase = 1;
        ...
    } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
        phase = 2;
        ...
    }
    ...
    if (collectionPhases[col].salesOption == 3) {
        ...
    }
}
```

It can be very difficult to understand what phase = 1 means and so on. Consider implementing ENUM for example:
```solidity
enum Phase{ ALLOWLIST, PUBLIC }
```
and then setting phases like
```solidity
phase = Phase.PUBLIC;
```

it offers better readability. Consider doing the same for the ```salesOption```

## Check that collections are not the same in ```burnToMint```
```solidity
function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o)
    public
    payable
{
    require(_burnCollectionID != _mintCollectionID, "cannot mint to the same colleciton");
    require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
    ...
}
```

## No check for different addresses in ```proposePrimaryAddressesAndPercentages```
```_primaryAdd1```, ```_primaryAdd2``` and ```_primaryAdd3``` can be the same. If it's not intended to be this way check that they are different using a require statement.

```solidity
function proposePrimaryAddressesAndPercentages(
    uint256 _collectionID,
    address _primaryAdd1,
    address _primaryAdd2,
    address _primaryAdd3,
    uint256 _add1Percentage,
    uint256 _add2Percentage,
    uint256 _add3Percentage
) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {
    require(collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
    require(
        _add1Percentage + _add2Percentage + _add3Percentage
            == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage,
        "Check %"
    );
    collectionArtistPrimaryAddresses[_collectionID].primaryAdd1 = _primaryAdd1;
    collectionArtistPrimaryAddresses[_collectionID].primaryAdd2 = _primaryAdd2;
    collectionArtistPrimaryAddresses[_collectionID].primaryAdd3 = _primaryAdd3;
    collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
    collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
    collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
    collectionArtistPrimaryAddresses[_collectionID].status = false;
}
```

## Double check that the protocol doesn't send funds to ```address(0)``` in ```emergencyWithdraw```

```solidity
function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
    uint256 balance = address(this).balance;
    address admin = adminsContract.owner();
+   require(admin != address(0), "sending to address(0)");

    (bool success,) = payable(admin).call{value: balance}("");
    emit Withdraw(msg.sender, success, balance);
}
```

## Consider using a bps representation of the percentage split to allow for more precision
```artistPercentage```and ```teamPercentage``` are represented from ```0 to 100```. Consider representing 100% from ```0 to 10_000``` to add more precision on splits

## Make sure the auction end time is in the future
Maybe consider adding a ```MIN_DURATION``` and ```MAX_DURATION``` also.

```solidity
function mintAndAuction(
    address _recipient,
    string memory _tokenData,
    uint256 _saltfun_o,
    uint256 _collectionID,
    uint256 _auctionEndTime
) public FunctionAdminRequired(this.mintAndAuction.selector) {
    require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
+   require(_auctionEndTime > block.timestamp);
    ...
}
```

## ```_burnCollectionID``` in the ```initializeExternalBurnOrSwap```and ```burnOrSwapExternalToMint``` serves no purpose
```solidity
function initializeExternalBurnOrSwap(
    address _erc721Collection,
    uint256 _burnCollectionID,
    uint256 _mintCollectionID,
    uint256 _tokmin,
    uint256 _tokmax,
    address _burnOrSwapAddress,
    bool _status
) public FunctionAdminRequired(this.initializeExternalBurnOrSwap.selector) {
    bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection, _burnCollectionID));
    require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");
    burnExternalToMintCollections[externalCol][_mintCollectionID] = _status;
    burnOrSwapAddress[externalCol] = _burnOrSwapAddress;
    burnOrSwapIds[externalCol][0] = _tokmin;
    burnOrSwapIds[externalCol][1] = _tokmax;
}

function burnOrSwapExternalToMint(
    address _erc721Collection,
    uint256 _burnCollectionID,
    uint256 _tokenId,
    uint256 _mintCollectionID,
    string memory _tokenData,
    bytes32[] calldata merkleProof,
    uint256 _saltfun_o
) public payable {
    bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection, _burnCollectionID));
    require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
    require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
    address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
    if (msg.sender != ownerOfToken) {
        bool isAllowedToMint;
        isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(
            ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1
        )
            || dmc.retrieveGlobalStatusOfDelegation(
                ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2
            );
        if (isAllowedToMint == false) {
            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1)
                || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);
        }
        require(isAllowedToMint == true, "No delegation");
    }
    require(
        _tokenId >= burnOrSwapIds[externalCol][0] && _tokenId <= burnOrSwapIds[externalCol][1],
        "Token id does not match"
    );
    IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
    uint256 col = _mintCollectionID;
    address mintingAddress;
    uint256 phase;
    string memory tokData = _tokenData;
    if (
        block.timestamp >= collectionPhases[col].allowlistStartTime
            && block.timestamp <= collectionPhases[col].allowlistEndTime
    ) {
        phase = 1;
        bytes32 node;
        node = keccak256(abi.encodePacked(_tokenId, tokData));
        mintingAddress = ownerOfToken;
        require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), "invalid proof");
    } else if (
        block.timestamp >= collectionPhases[col].publicStartTime
            && block.timestamp <= collectionPhases[col].publicEndTime
    ) {
        phase = 2;
        mintingAddress = ownerOfToken;
        tokData = '"public"';
    } else {
        revert("No minting");
    }
    uint256 collectionTokenMintIndex;
    collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
    require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
    require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
    uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
    gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);

    collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
}
```

It doesn't make sense to use ```_burnCollectionID``` because we are already using the ```_erc721Collection``` to indicate the erc721 collection we want to burn tokens from. To me, it seems like a redundant parameter and it only makes things more complex. Use only the ```_erc721Collection``` parameter.

```solidity
bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection));
```
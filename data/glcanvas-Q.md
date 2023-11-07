# [L-01] Contracts not in CamelCase

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L13
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L18

By convention contract names should be in CamelCase.

# [L-02] Contract names doesn't match file names

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L19
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L18
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L18
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L20

By convention names should match file names.

# [L-03] Store address only without Interface

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L21-L22

```solidity
    address gencore;
INextGenCore public gencoreContract;

```

Instead of `gencoreContract` you can use just `INextGenCore(gencore)`.

# [L-04] Lack of checking is admin contract

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L46

Add next
require: `require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");`

# [L-05] Typos

* collectionAdditonalDataStructure -> collectionAdditionalDataStructure
* tokends -> tokenIds
* retrievewereDataAdded -> retrieveWereDataAdded
* exponetialy -> exponentially
* decrase -> decrease
* airDropTokens -> airDropToken

# [L-06] Make public variable instead of get-view functions

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L57
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L438-L440
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L420-L422

Instead of creating view function for each structure just make it public.
Same for all other cases.

# [L-07] Use constants instead of numbers

* For `0x1B1289E34Fe05019511d7b436a5138F361904df0` use constant declaration for clarity;
* `10000000000` replace to `MAX_COLLECTION_SUPPLY`;
* `100` replace to `ONE_HUNDRED_PERCENT`;
* `phase` can be ether enum or `ALLOW_LIST_PHASE` and `PUBLIC_SALE_PHASE`;
* `0x0000000000000000000000000000000000000000` just `address(0)`;
* `0x8888888888888888888888888888888888888888` replace to `GLOBAL_CONFIG_ADDRESS` or smth similar;
* `salesOption` -- create enum or use constants instead of numbers `1`, `2`, `3`.

# [L-08] Bad function

```solidity
    function updateCollectionInfo(uint256 _collectionID, string memory _newCollectionName, string memory _newCollectionArtist, string memory _newCollectionDescription, string memory _newCollectionWebsite, string memory _newCollectionLicense, string memory _newCollectionBaseURI, string memory _newCollectionLibrary, uint256 _index, string[] memory _newCollectionScript) public CollectionAdminRequired(_collectionID, this.updateCollectionInfo.selector) {
    require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
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

This is anti-pattern, when one function do significantly different. better divide it into three
different functions.

# [L-09] Remove indexed for funds

```solidity
    event PayArtist(address indexed _add, bool status, uint256 indexed funds);
event PayTeam(address indexed _add, bool status, uint256 indexed funds);
event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```

Because no ant reason to find by funds amount.

# [L-10] Use atomic operation to find next free index

Now in minter contract present a lot of lines like this:

```solidity
collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```

This is anti-pattern because it's too easy to make a mistake.
Better create view function in Core contract, which will returns next free index, or rewert if
Collection sold-out.

# [L-11] Revert if collection full

```solidity
function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
    require(msg.sender == minterContract, "Caller is not the Minter Contract");
    collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
    if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
        _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
        tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
    }
}

function mint(uint256 mintIndex, address _mintingAddress, address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {
    require(msg.sender == minterContract, "Caller is not the Minter Contract")
    collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
    if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
        _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
        if (phase == 1) {
            tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;
        } else {
            tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;
        }
    }
}

function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
    require(msg.sender == minterContract, "Caller is not the Minter Contract");
    require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved"); 
    collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1; 
    if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
        _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
        // burn token
        _burn(_tokenId);
        burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
    }
}
```

Now
this `collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply`
expression always true, but due to code weaknesses this might be break, and circulation will be
bigger than total supply.
Add require condition, to prevent such cases.

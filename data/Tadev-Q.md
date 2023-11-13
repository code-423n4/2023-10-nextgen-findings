## [L‑01] `requestRandomWords()` function in RandomizerRNG contract shouldn't be payable and should be internal or private.

The RandomizerRNG includes the following 2 functions : 

```
 function requestRandomWords(uint256 tokenid, uint256 _ethRequired) public payable {
        require(msg.sender == gencore);
        uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
        tokenToRequest[tokenid] = requestId;
        requestToToken[requestId] = tokenid;
    }

  function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
        require(msg.sender == gencore);
        tokenIdToCollection[_mintIndex] = _collectionID;
        requestRandomWords(_mintIndex, ethRequired);
    }
```

Given the documentation and the implementation of NextGenCore contract, there is only one valid path : 

- NextGenCore calls `calculateTokenHash()` function without sending eth. NextGenCore or any other NextGen contract doesn't make any direct call to `requestRandomWords()` function. Currently, `requestRandomWords()` function also requires that `msg.sender == gencore` (just like `calculateTokenHash()` function , which is redondant. `requestRandomWords()` function should be a private function (or internal, at least) while removing the unnecessary check for msg.sender.

- `calculateTokenHash()` function internally calls `requestRandomWords()` function, without sending ETH. There is actually no way to call `requestRandomWords()` function while sending ETH, this is why `payable` keyword should be removed. The final call to `arrngController.requestRandomWords()` function will use ETH balance of the Randomizer contract to send `_ethRequired` ETH value. 

That being said, I propose to modify these functions as follows : 

```
  function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) external {
        require(msg.sender == gencore); // ideally, use a custom error to save gas
        tokenIdToCollection[_mintIndex] = _collectionID;
        requestRandomWords(_mintIndex, ethRequired);
    }

 function requestRandomWords(uint256 tokenid, uint256 _ethRequired) private {
        uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
        tokenToRequest[tokenid] = requestId;
        requestToToken[requestId] = tokenid;
    }
```


## [L‑02] There is a situation in which `reservedMinTokensIndex` value is greater than `reservedMaxTokensIndex` value in NextGenCore contract storage, which is not supposed to happen.

`reservedMinTokensIndex` and `reservedMaxTokensIndex` are both members of the `collectionAdditonalDataStructure` struct. They are supposed to bound all available tokens for a collection.
In the following scenario:
- A new collection is created with `createCollection()` function
- The collection admin (or higher credential) calls `setCollectionData()`, passing 0 as value for `_collectionTotalSupply` variable. The result will be : 
```
collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + 0 - 1;
reservedMinTokensIndex = reservedMaxTokensIndex + 1
```
This situation could be avoided by adding a check that `_collectionTotalSupply` is strictly greater than 0.


## [L‑03] It is possible to make 2 calls to `setCollectionData()` function in NextGenCore contract and modify `collectionTotalSupply` value in the second one.

Documentation clearly says : 
"After its initial call, the setCollectionData() function can be called to update the artists eth address, the max purchases during public minting and the final supply time."

Nevertheless, if you firstly call `setCollectionData()` function with a value of 0 for `_collectionTotalSupply`, you can make another call to set `collectionTotalSupply` value. 

To respect the specification, I would recommend to do the same thing as in [L‑02]: adding a check at the beginning of `setCollectionData()` function to make sure that `_collectionTotalSupply` input is strictly greater than 0.


## [L‑04] Unnecessary and redondant check in `mint()` function in NextGenMinterContract

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L223-L224

During the public mint period (phase == 2), a call to the `mint()` function will trigger a check that the number of tokens the user requested to mint is not too much. In order to do that, the function needs to compare: 
- The number of tokens the user is willing to mint (`_numberOfTokens`) + the number of tokens already minted by the user
- `gencore.viewMaxAllowance(col)`, which returns `maxCollectionPurchases`

This means only the following check is required : 
```
require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
```
The check executed just before is clearly redondant and not sufficient, as it only checks that `_numberOfTokens` is lower than `maxCollectionPurchases`:
```
require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
```

I suggest to keep only the 2nd check, and delete the first check. This would also save gas.


## [L‑05] Add a safety check in `setCollectionData()` function in NextGenCore contract to make sure `_maxCollectionPurchases < _collectionTotalSupply` 

Currently, there is no sanity check for all uint256 values that are passed to `setCollectionData()` function (except `_collectionTotalSupply <= 10000000000`).
This means it could be possible to create a collection with `_maxCollectionPurchases < _collectionTotalSupply`, which is not supposed to happen.

It could be a good idea to add the following check (or ideally, with a custom error): 
```
require(_maxCollectionPurchases <= _collectionTotalSupply, "aberrant max value");
````


## [L‑06] There is no check in `setCollectionPhases()` function in NextGenMinterContract to make sure that `_allowlistStartTime < _allowlistEndTime` and `_publicStartTime < _publicEndTime`.

`setCollectionPhases()` function can be called with wrong values for  `_allowlistStartTime`, `_allowlistEndTime`, `_publicStartTime` and `_publicEndTime`. This could lead to a collection not working correctly, with `mint()`, `burnToMint()` and `burnOrSwapExternalToMint()` functions reverting, and `getPrice()` function not working correctly.

It could be a good idea to a check to make sure `_allowlistStartTime > _allowlistEndTime` and `_publicStartTime > _publicEndTime`.


## [L‑07] Lack of precision in `proposePrimaryAddressesAndPercentages()` and `proposeSecondaryAddressesAndPercentages` functions for percentages

All percentage system uses 100 as the total value than can be splitted.

Both functions use this check : 
```
    require(
            _add1Percentage + _add2Percentage + _add3Percentage
                == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage,
            "Check %"
        );
```
This means all percentages could only be integer value, as `artistPercentage` is an integer between 0 and 100. It is not possible to choose, for example,  `_add1Percentage` representing 6.5%. This could be improved to allow more granularity in the way funds are split between artist's addresses and the team's addresses. Using 1000 or 10 000 as a base for the percentage system would allow this.


## [L‑08] Minting period (either allowlist phase or public phase) could start although artist didn't sign the collection

As precised by the sponsor in the discord channel of the contest:
"artists will need to sign the collection before even minting starts, this will be an agreement between team and artist".

But currently, minting period could start although artist didn't sign the collection, there is no check preventing this from happening.

Therefore, I suggest to add a getter function in NextGenCore contract:
```
   function hasArtistSigned(uint256 _collectionID) public view returns (bool) {
        return artistSigned[_collectionID];
    }
```
and a check in `setCollectionCosts()` function in NextGenMinterContract :
```
require(gencore.hasArtistSigned(_collectionID), "artist didn't sign"); // custom error ideally
```
This would ensure the whole minting part of the process of emitting a collection will begin only after artist signed it.

INextGenCore interface should also be modified to add the function declaration: 
```
    function hasArtistSigned(uint256 _collectionID) external view returns (bool);
```

## [N‑01] It is not clear how RandomizerRNG contract will be funded in order to be able to call `requestRandomWords()` function

RandomizerRNG contract, contrary to other randomizers, needs to send ETH when calling `arrngController.requestRandomWords()` function, in order to request and then obtain random numbers to generate the unique hash of the token.

Currently, there are 2 payable functions : `requestRandomWords()` and `receive()` . 
`requestRandomWords()` is problematic as the only way to call it is through gencore by calling `calculateTokenHash()` function, which is not payable. Therefore, it is only possible to send ether to this contract is to ''raw'' send ether, calling `receive()` function.

In this configuration, there is no transparency in how the contract will be funded, by who, and users cannot be sure that the randomizer will always work, as it depends on its funding which is not programmatically determined.


## [N‑02] `minterContract` variable type in NextGenCore contract and `gencore` variable type in AuctionDemo contract should be changed to be consistent

NextGenCore declares in its storage 2 variables related to other contracts : 
```
    INextGenAdmins private adminsContract;
    address public minterContract;
```
Both are named as "contract", while their type is not consistent. 
I recommend to replace `address public minterContract` with `IMinterContract public minterContract)`, and modify subsequent code where this variable is used (either to wrap it or to unwrap it).

In AuctionDemo contract, state variables are declared as follows : 
```
IMinterContract public minter;
INextGenAdmins public adminsContract;
address gencore;
```
`address gencore` should be replaced by `INextGenCore public gencoreContract` or `IERC721 public gencore` (after importing INextGenCore if the first solution is chosen). This would increase consistency. This variable is used only twice, and is wrapped each time into an `IERC721` type. Changing the type of the variable would prevent from doing this unnecessary wrapping.


## [N‑03] Order of parameters in `setCollectionCosts()` function is not consistent with `collectionPhasesDataStructure` struct. The same applies for the order of return variables in `retrieveCollectionPhases()` function.

`collectionPhasesDataStructure` struct is represented as follows : 
```
    struct collectionPhasesDataStructure {
        uint256 allowlistStartTime;
        uint256 allowlistEndTime;
        uint256 publicStartTime;
        uint256 publicEndTime;
        bytes32 merkleRoot;
        uint256 collectionMintCost;
        uint256 collectionEndMintCost;
        uint256 timePeriod;
        uint256 rate;
        uint8 salesOption;
        address delAddress;
    }
```

`setCollectionCosts` function actually switched `timePeriod` and `rate` position : 

```
   function setCollectionCosts(
        uint256 _collectionID,
        uint256 _collectionMintCost,
        uint256 _collectionEndMintCost,
        uint256 _rate,
        uint256 _timePeriod,
        uint8 _salesOption,
        address _delAddress
    ) 
```
`retrieveCollectionPhases()` function returns variables of the struct in the following order: 
```
collectionPhases[_collectionID].allowlistStartTime,
collectionPhases[_collectionID].allowlistEndTime,
collectionPhases[_collectionID].merkleRoot,
collectionPhases[_collectionID].publicStartTime,
collectionPhases[_collectionID].publicEndTime
```
`merkleroot` variable should be returned after `publicEndTime` to increase consistency.

This is error-prone, and the order of parameters should be modified in `setCollectionCosts()` function, as well as the order of return variables in `retrieveCollectionPhases()` function.
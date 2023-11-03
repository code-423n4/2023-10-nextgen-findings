## [L‑01] It is not clear how RandomizerRNG contract will be funded in order to be able to call `requestRandomWords()` function

RandomizerRNG contract, contrary to other randomizers, needs to send ETH when calling `arrngController.requestRandomWords()` function, in order to request and then obtain random numbers to generate the unique hash of the token.

Currently, there are 2 payable functions : `requestRandomWords()` and `receive()` . 
`requestRandomWords()` is problematic as the only way to call it is through gencore by calling `calculateTokenHash()` function, which is not payable. Therefore, it is only possible to send ether to this contract is to ''raw'' send ether, calling `receive()` function.

In this configuration, there is no transparency in how the contract will be funded, by who, and users cannot be sure that the randomizer will always work, as it depends on its funding which is not programmatically determined.


## [L‑02] `requestRandomWords()` function in RandomizerRNG contract shouldn't be payable and should be internal or private

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

Given the documentation and implementation of NextGenCore contract, there is only one valid path : 

- NextGenCore calls `calculateTokenHash()` function without sending value. NextGenCore doesn't make any direct call to `requestRandomWords()` function. As `requestRandomWords()` also requires that `msg.sender == gencore`, it should be a private function (or internal, at least) while removing the check for msg.sender.

- `calculateTokenHash()` function internally calls `requestRandomWords()` function, without sending ETH. There is actually no way to call `requestRandomWords()` function while sending ETH, thats why `payable` should be removed. The final call to `arrngController.requestRandomWords()` function will use ETH balance of the contract to send `_ethRequired` ETH value. 

That being said, I propose to modify these functions as follows : 

```
  function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) external {
        require(msg.sender == gencore); // use a custom error to save gas
        tokenIdToCollection[_mintIndex] = _collectionID;
        requestRandomWords(_mintIndex, ethRequired);
    }

 function requestRandomWords(uint256 tokenid, uint256 _ethRequired) internal {
        uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
        tokenToRequest[tokenid] = requestId;
        requestToToken[requestId] = tokenid;
    }
```


## [L‑03] `emergencyWithdraw()` function of RandomizerRNG contract can be called by only one FunctionAdmin, who is able to withdraw all funds to the wallet of the owner of NextGenAdmins contract.

As it is currently designed, `emergencyWithdraw()` function introduces a centralization risk, as any admin authorized to call this function could execute a withdrawal of all funds to the wallet of the owner of NextGenAdmins contract. This way, any allowed admin could decide by himself to block the contract ands its ability to generate random hashes and send them to the Core contract.

It would be a good idea to update the owner to be a DAO, once deployed. This would allow more granularity is the way withdrawal of funds can be executed.


## [L‑04] There is a situation in which `reservedMinTokensIndex` value is greater than `reservedMaxTokensIndex` value in NextGenCore contract storage

`reservedMinTokensIndex` and `reservedMaxTokensIndex` are both members of the `collectionAdditonalDataStructure` struct. They are supposed to bound all available tokens for a collection.
In the following scenario  :
- A new collection is created with `createCollection()` function
- The collection admin (or higher credential) calls `setCollectionData()`, passing 0 as value for `_collectionTotalSupply` variable. The result will be : 

```
collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + 0 - 1;

reservedMinTokensIndex = reservedMaxTokensIndex + 1
```
This situation could be avoided by adding a check that `_collectionTotalSupply` is strictly greater than 0.


## [L‑05] It is possible to make 2 calls to `setCollectionData()` function in NextGenCore contract and modify `collectionTotalSupply` value in the second one.

Documentation clearly says : 
"After its initial call, the setCollectionData() function can be called to update the artists eth address, the max purchases during public minting and the final supply time."

Nevertheless, if you firstly call `setCollectionData()` function with a value of 0 for `_collectionTotalSupply`, you can make another call to set `collectionTotalSupply` value. 

To respect the specification, i would recommend to do the same thing as in [L‑04] : adding a check at the beginning of `setCollectionData()` function that `_collectionTotalSupply` input is strictly greater than 0.


## [L‑06] Unnecessary and redondant check in `mint()` function in MinterContract

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L223-L224

During the public mint period (phase ==2), a call to the `mint()` function will trigger a check that the number of tokens the user requested to mint is not too much. In order to do that, the function needs to compare : 
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

I suggest to keep only the 2nd check, and delete the first check.


## [L‑07]








|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[G-1](#g-1--use-assembly-to-write-address-storage-values)] | use assembly to write address storage values | Gas Optimization | Informational | 8 |
| [[G-2](#g-2--Use-do-while-loops-instead-of-for-loops)] | Use do while loops instead of for loops | Gas Optimization | Informational | 6 |
| [[G-3](#g-3--newcollectionindex-value-can-be-set-outside-the-constructor)] | newCollectionIndex value can be set outside the constructor  | Gas Optimization | Informational | 1 |
| [[G-4](#g-4--no-need-to-check--true)] |  no need to check == true   | Gas Optimization | Informational | 26 |
| [[G-5](#g-5--with-assembly-call-transfer-can-be-done-gas-optimized)] |  With assembly .call transfer can be done gas-optimized   | Gas Optimization | Informational | 1 |
| [[G-6](#g-6--arrayindex--amount-is-cheaper-than-arrayindex--arrayindex--amount)] |  array[index] += amount is cheaper than array[index] = array[index] + amount   | Gas Optimization | Informational | 8 |
| [[G-7](#g-7--possible-optimizations-in-mint-function)] | Possible Optimizations in `mint` function  | Gas Optimization | Informational | 1 |


## **G-1** | Use assembly to write address storage values 


By using assembly, we can directly interact with the storage slot of the storedAddress variable, allowing us to efficiently write and read address values in storage.

```diff
diff --git a/MinterContract.sol b/aMinterContract.sol
index df50841..6c8472d 100644
--- a/MinterContract.sol
+++ b/aMinterContract.sol
@@ -126,10 +126,16 @@ contract NextGenMinterContract is Ownable {
     event Withdraw(address indexed _add, bool status, uint256 indexed funds);
 
     // constructor
+    //@audit Use assembly to write address storage values 
     constructor (address _gencore, address _del, address _adminsContract) {
-        gencore = INextGenCore(_gencore);
-        dmc = IDelegationManagementContract(_del);
-        adminsContract = INextGenAdmins(_adminsContract);
+        //INextGenCore _gencore = INextGenCore(_gencore);
+        //dmc = IDelegationManagementContract(_del);
+        //adminsContract = INextGenAdmins(_adminsContract);
+        assembly{
+            sstore(gencore.slot,_gencore)
+            sstore(dmc.slot,_del)
+            sstore(adminsContract.slot,_adminsContract)
+        }
     }
 
     // certain functions can only be called by an admin or the artist
@@ -446,14 +452,20 @@ contract NextGenMinterContract is Ownable {
     // function to update core contract
 
     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
-        gencore = INextGenCore(_gencore);
+        //gencore = INextGenCore(_gencore);
+        assembly{
+            sstore(gencore.slot,_gencore)
+        }
     }
 
     // function to update admin contract
 
     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
         require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
-        adminsContract = INextGenAdmins(_newadminsContract);
+        //adminsContract = INextGenAdmins(_newadminsContract);
+        assembly{
+            sstore(adminsContract.slot,_newadminsContract)
+        }
     }
 
     // function to withdraw any balance from the smart contract
```

[MinterContract](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L129-L133)

[MinterContract](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L448-L450)

[MinterContract](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L454-L457)


```diff
diff --git a/NextGenCore.sol b/aNextGenCore.sol
index 6d294ed..a70cd00 100644
--- a/NextGenCore.sol
+++ b/aNextGenCore.sol
@@ -106,7 +106,10 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
 
     // smart contract constructor
     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
-        adminsContract = INextGenAdmins(_adminsContract);
+        //adminsContract = INextGenAdmins(_adminsContract);
+        assembly{
+            sstore(adminsContract.slot,_adminsContract)
+        }
         newCollectionIndex = newCollectionIndex + 1;
         _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
     }
@@ -314,7 +317,10 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
 
     function addMinterContract(address _minterContract) public FunctionAdminRequired(this.addMinterContract.selector) { 
         require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");
-        minterContract = _minterContract;
+        //minterContract = _minterContract;
+        assembly{
+            sstore(minterContract.slot,_minterContract)
+        }
     }
 
     // function to update admin contract
@@ -322,6 +328,9 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
         require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
         adminsContract = INextGenAdmins(_newadminsContract);
+        assembly{
+            sstore(adminsContract.slot,_newadminsContract)
+        }
     }
 
     // function to update default royalties
```
[NextGenCore#108-112](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L108-L112)

[NextGenCore#315-318](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L315-L318)

[NextGenCore#322-325](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L322-L325)

> Gas diff


| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `NextGenCore.addMinterContract` | 58433 | 58367 | -66 | -0.11% |
| `NextGenCore` | 5501759 | 5488798 | -12961 | -0.24% |
| `NextGenMinterContract` | 5454331 | 5440815 | -13516 | -0.25% |



## **G-2** | Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration. 

```diff
diff --git a/NextGenCore.sol b/aNextGenCore.sol
index f61d6f2..681a566 100644
--- a/NextGenCore.sol
+++ b/aNextGenCore.sol
@@ -279,12 +279,17 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // function to add a thumbnail image
 
     function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
-        for (uint256 x; x < _tokenId.length; x++) {
+        uint256 x;
+        do{
+        //for (uint256 x; x < _tokenId.length; x++) {
             require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
             _requireMinted(_tokenId[x]);
             tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
             tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
-        }
+            unchecked{
+                ++x;
+            }
+        }while(x < _tokenId.length);
     }
 
     // freeze collection
@@ -450,9 +455,14 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     function retrieveGenerativeScript(uint256 tokenId) public view returns(string memory){
         _requireMinted(tokenId);
         string memory scripttext;
-        for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
+        uint256 i;
+        do{
+        //for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
             scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i])); 
-        }
+            unchecked{
+                ++i;
+            }
+        }while(i<collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length);
         return string(abi.encodePacked("let hash='",Strings.toHexString(uint256(tokenToHash[tokenId]), 32),"';let tokenId=",tokenId.toString(),";let tokenData=[",tokenData[tokenId],"];", scripttext));
     }
 

```

[NextGenCore#L282](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L282-287)

[NextGenCore#L453](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L453-455)



```diff
diff --git a/MinterContract.sol b/aMinterContract.sol
index df6f38d..dbf7ec9 100644
--- a/MinterContract.sol
+++ b/aMinterContract.sol
@@ -181,14 +181,23 @@ contract NextGenMinterContract is Ownable {
     function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
         uint256 collectionTokenMintIndex;
-        for (uint256 y=0; y< _recipients.length; y++) {
+        uint256 y;
+        do{
+            //for (uint256 y=0; y< _recipients.length; y++) {
             collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
             require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
-            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
+            uint256 i;
+            do{//for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
                 uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                 gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
+                unchecked{
+                    ++i;
+                }
+            }while(i<_numberOfTokens[y]);
+            unchecked{
+                ++y;
             }
-        }
+        }while(y < _recipients.length);
     }
 
     // mint function
@@ -231,10 +240,15 @@ contract NextGenMinterContract is Ownable {
         collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
         require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
-        for(uint256 i = 0; i < _numberOfTokens; i++) {
+        uint256 i;
+        do{
+        //for(uint256 i = 0; i < _numberOfTokens; i++) {
             uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
             gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
-        }
+            unchecked{
+                ++i;
+            }
+        }while(i<_numberOfTokens);
         collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
         // control mechanism for sale option 3
         if (collectionPhases[col].salesOption == 3) {

```

[MinterContract#L184](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L184-L191)

[MinterContract#L234](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L234-L237)


```diff
diff --git a/NextGenAdmins.sol b/aNextGenAdmins.sol
index dc68ddd..f8dc971 100644
--- a/NextGenAdmins.sol
+++ b/aNextGenAdmins.sol
@@ -48,9 +48,14 @@ contract NextGenAdmins is Ownable{
     // function to register batch functions admin
 
     function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
-        for (uint256 i=0; i<_selector.length; i++) {
+        uint256 i;
+        do{
+        //for (uint256 i=0; i<_selector.length; i++) {
             functionAdmin[_address][_selector[i]] = _status;
-        }
+            unchecked {
+                ++i;
+            }
+        }while(i<_selector.length);
     }

```
[NextGenAdmins#L51](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L51-L53)

> Gas diff 

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `NextGenMinterContract.airDropTokens` | 744940 | 744382 | -558 | -0.07% |
| `NextGenMinterContract.mint` | 404474 | 404351 | -123 | -0.03% |
| `NextGenAdmins` | 582355 | 571365 | -10990 | -1.89% |
| `NextGenCore` | 5501759 | 5493766 | -7993 | -0.15% |
| `NextGenMinterContract` | 5454331 | 5445464 | -8867 | -0.16% |

## **G-3** | newCollectionIndex value can be set outside the constructor

newCollectionIndex is 0 set it to 1 in constructor west gas ! we can change it before deploy 

```diff
diff --git a/NextGenCore.sol b/aNextGenCore.sol
index 6d294ed..f60def8 100644
--- a/NextGenCore.sol
+++ b/aNextGenCore.sol
@@ -23,7 +23,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     using Strings for uint256;
 
     // declare variables
-    uint256 public newCollectionIndex;
+    uint256 public newCollectionIndex = 1;
 
     // collectionInfo struct declaration
     struct collectionInfoStructure {
@@ -107,7 +107,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // smart contract constructor
     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
         adminsContract = INextGenAdmins(_adminsContract);
-        newCollectionIndex = newCollectionIndex + 1;
+        //newCollectionIndex = newCollectionIndex + 1;
         _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
     }

```
[NextGenCore.sol#L26](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L26)

> Gas diff 

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `NextGenCore` | 5501759 | 5500900 | -859 | -0.02% |


## **G-4** | no need to check == true 

Don't compare boolean expressions to boolean literals

```diff
diff --git a/MinterContract.sol b/aMinterContract.sol
index df6f38d..5245b0f 100644
--- a/MinterContract.sol
+++ b/aMinterContract.sol
@@ -134,28 +134,28 @@ contract NextGenMinterContract is Ownable {
 
     // certain functions can only be called by an admin or the artist
     modifier ArtistOrAdminRequired(uint256 _collectionID, bytes4 _selector) {
-      require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
+      require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector)  || adminsContract.retrieveGlobalAdmin(msg.sender) , "Not allowed");
       _;
     }
 
     // certain functions can only be called by a global or function admin
 
     modifier FunctionAdminRequired(bytes4 _selector) {
-      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
+      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector)  || adminsContract.retrieveGlobalAdmin(msg.sender)  , "Not allowed");
       _;
     }
 
     // certain functions can only be called by a collection, global or function admin
 
     modifier CollectionAdminRequired(uint256 _collectionID, bytes4 _selector) {
-      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
+      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID)  || adminsContract.retrieveFunctionAdmin(msg.sender, _selector)  || adminsContract.retrieveGlobalAdmin(msg.sender) , "Not allowed");
       _;
     }
 
     // function to add a collection's minting costs
 
     function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
-        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
+        require(gencore.retrievewereDataAdded(_collectionID) , "Add data");
         collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
         collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
         collectionPhases[_collectionID].rate = _rate;
@@ -168,7 +168,7 @@ contract NextGenMinterContract is Ownable {
     // function to add a collection's start/end times and merkleroot
 
     function setCollectionPhases(uint256 _collectionID, uint _allowlistStartTime, uint _allowlistEndTime, uint _publicStartTime, uint _publicEndTime, bytes32 _merkleRoot) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
-        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
+        require(setMintingCosts[_collectionID] , "Set Minting Costs");
         collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
         collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
         collectionPhases[_collectionID].merkleRoot = _merkleRoot;
@@ -179,7 +179,7 @@ contract NextGenMinterContract is Ownable {
     // airdrop function
     
     function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
-        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
+        require(gencore.retrievewereDataAdded(_collectionID) , "Add data");
         uint256 collectionTokenMintIndex;
         for (uint256 y=0; y< _recipients.length; y++) {
             collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
@@ -194,7 +194,7 @@ contract NextGenMinterContract is Ownable {
     // mint function
 
     function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
-        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
+        require(setMintingCosts[_collectionID] , "Set Minting Costs");
         uint256 col = _collectionID;
         address mintingAddress;
         uint256 phase;
@@ -208,7 +208,7 @@ contract NextGenMinterContract is Ownable {
                 if (isAllowedToMint == false) {
                 isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);    
                 }
-                require(isAllowedToMint == true, "No delegation");
+                require(isAllowedToMint , "No delegation");
                 node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));
                 require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");
                 mintingAddress = _delegator;
@@ -256,7 +256,7 @@ contract NextGenMinterContract is Ownable {
     // burn to mint function (does not require contract approval)
 
     function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
-        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
+        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] , "Initialize burn");
         require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
         require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
         // minting new token
@@ -274,7 +274,7 @@ contract NextGenMinterContract is Ownable {
     // mint and auction
     
     function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
-        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
+        require(gencore.retrievewereDataAdded(_collectionID) , "Add data");
         uint256 collectionTokenMintIndex;
         collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
@@ -306,7 +306,7 @@ contract NextGenMinterContract is Ownable {
     // function to initialize burn to mint for NextGen collections
 
     function initializeBurn(uint256 _burnCollectionID, uint256 _mintCollectionID, bool _status) public FunctionAdminRequired(this.initializeBurn.selector) { 
-        require((gencore.retrievewereDataAdded(_burnCollectionID) == true) && (gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");
+        require((gencore.retrievewereDataAdded(_burnCollectionID) ) && (gencore.retrievewereDataAdded(_mintCollectionID) ), "No data");
         burnToMintCollections[_burnCollectionID][_mintCollectionID] = _status;
     }
 
@@ -314,7 +314,7 @@ contract NextGenMinterContract is Ownable {
 
     function initializeExternalBurnOrSwap(address _erc721Collection, uint256 _burnCollectionID, uint256 _mintCollectionID, uint256 _tokmin, uint256 _tokmax, address _burnOrSwapAddress, bool _status) public FunctionAdminRequired(this.initializeExternalBurnOrSwap.selector) { 
         bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
-        require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");
+        require((gencore.retrievewereDataAdded(_mintCollectionID) ), "No data");
         burnExternalToMintCollections[externalCol][_mintCollectionID] = _status;
         burnOrSwapAddress[externalCol] = _burnOrSwapAddress;
         burnOrSwapIds[externalCol][0] = _tokmin;
@@ -325,8 +325,8 @@ contract NextGenMinterContract is Ownable {
 
     function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
         bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
-        require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
-        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
+        require(burnExternalToMintCollections[externalCol][_mintCollectionID] , "Initialize external burn");
+        require(setMintingCosts[_mintCollectionID] , "Set Minting Costs");
         address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
         if (msg.sender != ownerOfToken) {
             bool isAllowedToMint;
@@ -334,7 +334,7 @@ contract NextGenMinterContract is Ownable {
             if (isAllowedToMint == false) {
             isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);    
             }
-            require(isAllowedToMint == true, "No delegation");
+            require(isAllowedToMint , "No delegation");
         }
         require(_tokenId >= burnOrSwapIds[externalCol][0] && _tokenId <= burnOrSwapIds[externalCol][1], "Token id does not match");
         IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
@@ -413,7 +413,7 @@ contract NextGenMinterContract is Ownable {
     // function to pay the artist
 
     function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
-        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
+        require(collectionArtistPrimaryAddresses[_collectionID].status , "Accept Royalties");
         require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
         require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
         uint256 royalties = collectionTotalAmount[_collectionID];
@@ -452,7 +452,7 @@ contract NextGenMinterContract is Ownable {
     // function to update admin contract
 
     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
-        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
+        require(INextGenAdmins(_newadminsContract).isAdminContract() , "Contract is not Admin");
         adminsContract = INextGenAdmins(_newadminsContract);
     }
 

```
[MinterContract.sol#L137](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L137)

[MinterContract.sol#L144](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L144)

[MinterContract.sol#L151](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L151)

[MinterContract.sol#L158](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L158)

[MinterContract.sol#L171](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L171)

[MinterContract.sol#L182](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L182)

[MinterContract.sol#L197](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L197)

[MinterContract.sol#L211](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L211)

[MinterContract.sol#L259](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L259)

[MinterContract.sol#L277](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L277)

[MinterContract.sol#L309](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L309)

[MinterContract.sol#L319](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L317)


[MinterContract.sol#L328](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L328)


[MinterContract.sol#L329](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L329)


[MinterContract.sol#L337](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L337)


[MinterContract.sol#L416](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L416)


[MinterContract.sol#L455](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L455)



```diff
diff --git a/NextGenAdmins.sol b/aNextGenAdmins.sol
index dc68ddd..712e465 100644
--- a/NextGenAdmins.sol
+++ b/aNextGenAdmins.sol
@@ -29,7 +29,7 @@ contract NextGenAdmins is Ownable{
 
     // certain functions can only be called by an admin
     modifier AdminRequired {
-      require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
+      require((adminPermissions[msg.sender] ) || (_msgSender()== owner()), "Not allowed");
       _;
     }

```
[NextGenAdmins#32](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L32)

```diff
diff --git a/NextGenCore.sol b/aNextGenCore.sol
index f61d6f2..8ba3dbc 100644
--- a/NextGenCore.sol
+++ b/aNextGenCore.sol
@@ -114,14 +114,14 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // certain functions can only be called by a global or function admin
 
     modifier FunctionAdminRequired(bytes4 _selector) {
-      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
+      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector)  || adminsContract.retrieveGlobalAdmin(msg.sender)  , "Not allowed");
       _;
     }
 
     // certain functions can only be called by a collection, global or function admin
 
     modifier CollectionAdminRequired(uint256 _collectionID, bytes4 _selector) {
-      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
+      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID)  || adminsContract.retrieveFunctionAdmin(msg.sender, _selector)  || adminsContract.retrieveGlobalAdmin(msg.sender) , "Not allowed");
       _;
     }
 
@@ -145,7 +145,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set
 
     function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
-        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
+        require((isCollectionCreated[_collectionID] ) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
         if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
             collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
             collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
@@ -168,7 +168,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // Add Randomizer contract on collection
 
     function addRandomizer(uint256 _collectionID, address _randomizerContract) public FunctionAdminRequired(this.addRandomizer.selector) {
-        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");
+        require(IRandomizer(_randomizerContract).isRandomizerContract() , "Contract is not Randomizer");
         collectionAdditionalData[_collectionID].randomizerContract = _randomizerContract;
         collectionAdditionalData[_collectionID].randomizer = IRandomizer(_randomizerContract);
     }
@@ -236,7 +236,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // function to update Collection Info
 
     function updateCollectionInfo(uint256 _collectionID, string memory _newCollectionName, string memory _newCollectionArtist, string memory _newCollectionDescription, string memory _newCollectionWebsite, string memory _newCollectionLicense, string memory _newCollectionBaseURI, string memory _newCollectionLibrary, uint256 _index, string[] memory _newCollectionScript) public CollectionAdminRequired(_collectionID, this.updateCollectionInfo.selector) {
-        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+        require((isCollectionCreated[_collectionID] ) && (collectionFreeze[_collectionID] == false), "Not allowed");
          if (_index == 1000) {
             collectionInfo[_collectionID].collectionName = _newCollectionName;
             collectionInfo[_collectionID].collectionArtist = _newCollectionArtist;
@@ -264,7 +264,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // function change metadata view 
 
     function changeMetadataView(uint256 _collectionID, bool _status) public CollectionAdminRequired(_collectionID, this.changeMetadataView.selector) { 
-        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+        require((isCollectionCreated[_collectionID] ) && (collectionFreeze[_collectionID] == false), "Not allowed");
         onchainMetadata[_collectionID] = _status;
     }
 
@@ -290,7 +290,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // freeze collection
 
     function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
-        require(isCollectionCreated[_collectionID] == true, "No Col");
+        require(isCollectionCreated[_collectionID] , "No Col");
         collectionFreeze[_collectionID] = true;
     }
 
@@ -313,14 +313,14 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // function to add a minter contract
 
     function addMinterContract(address _minterContract) public FunctionAdminRequired(this.addMinterContract.selector) { 
-        require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");
+        require(IMinterContract(_minterContract).isMinterContract() , "Contract is not Minter");
         minterContract = _minterContract;
     }
 
     // function to update admin contract
 
     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
-        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
+        require(INextGenAdmins(_newadminsContract).isAdminContract() , "Contract is not Admin");
         adminsContract = INextGenAdmins(_newadminsContract);
     }
 

```

[NextGenCore#L117](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L117)

[NextGenCore#L124](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L124)

[NextGenCore#L148](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148)

[NextGenCore#L171](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L171)

[NextGenCore#L239](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L239)

[NextGenCore#L267](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L267)

[NextGenCore#L293](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L293)

[NextGenCore#L316](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L316)

[NextGenCore#L323](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L323)

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `NextGenAdmins.registerCollectionAdmin` | 46728 | 46710 | -18 | -0.04% |
| `NextGenCore.addMinterContract` | 58433 | 58396 | -37 | -0.06% |
| `NextGenCore.addRandomizer` | 70696 | 70659 | -37 | -0.05% |
| `NextGenCore.createCollection` | 252555 | 252530 | -25 | -0.01% |
| `NextGenCore.setCollectionData` | 192459 | 192416 | -43 | -0.02% |
| `NextGenMinterContract.airDropTokens` | 744940 | 744903 | -37 | -0.00% |
| `NextGenMinterContract.mint` | 404474 | 404462 | -12 | -0.00% |
| `NextGenMinterContract.setCollectionCosts` | 147944 | 147894 | -50 | -0.03% |
| `NextGenMinterContract.setCollectionPhases` | 150002 | 149952 | -50 | -0.03% |
| `NextGenAdmins` | 582355 | 578465 | -3890 | -0.67% |
| `NextGenCore` | 5501759 | 5461721 | -40038 | -0.73% |
| `NextGenMinterContract` | 5454331 | 5399163 | -55168 | -1.01% |



## **G-5** | With assembly .call transfer can be done gas-optimized

`return` data `(bool success,)` has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.


```diff
diff --git a/MinterContract.sol b/aMinterContract.sol
index df6f38d..096d22f 100644
--- a/MinterContract.sol
+++ b/aMinterContract.sol
@@ -459,10 +459,14 @@ contract NextGenMinterContract is Ownable {
     // function to withdraw any balance from the smart contract
 
     function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
-        uint balance = address(this).balance;
+        uint _balance = address(this).balance;
         address admin = adminsContract.owner();
-        (bool success, ) = payable(admin).call{value: balance}("");
-        emit Withdraw(msg.sender, success, balance);
+        bool success;
+        //(bool success, ) = payable(admin).call{value: balance}("");
+        assembly{
+            success := call(gas(), admin, _balance, 0, 0, 0, 0)
+        }
+        emit Withdraw(msg.sender, success, _balance);
     }

```

[MinterContract.sol#L464](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L464)

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `NextGenMinterContract` | 5454331 | 5438769 | -15562 | -0.29% |


## **G-6** | array[index] += amount is cheaper than array[index] = array[index] + amount

When updating a value in an array with arithmetic, using array[index] += amount is cheaper than array[index] = array[index] + amount. This is because you avoid an additional mload when the array is stored in memory, and an sload when the array is stored in storage. This can be applied for any arithmetic operation including +=, -=,/=,*=,^=,&=, %=, <<=,>>=, and >>>=. This optimization can be particularly significant if the pattern occurs during a loop.


```diff
diff --git a/NextGenCore.sol b/aNextGenCore.sol
index f61d6f2..0efed4c 100644
--- a/NextGenCore.sol
+++ b/aNextGenCore.sol
@@ -177,10 +177,10 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     
     function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
         require(msg.sender == minterContract, "Caller is not the Minter Contract");
-        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
+        collectionAdditionalData[_collectionID].collectionCirculationSupply += 1;
         if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
             _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
-            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
+            tokensAirdropPerAddress[_collectionID][_recipient] += 1;
         }
     }
 
@@ -188,13 +188,13 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
 
     function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {
         require(msg.sender == minterContract, "Caller is not the Minter Contract");
-        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
+        collectionAdditionalData[_collectionID].collectionCirculationSupply += 1;
         if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
             _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
             if (phase == 1) {
-                tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;
+                tokensMintedAllowlistAddress[_collectionID][_mintingAddress] += 1;
             } else {
-                tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;
+                tokensMintedPerAddress[_collectionID][_mintingAddress] += 1;
             }
         }
     }
@@ -205,7 +205,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
         require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");
         require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");
         _burn(_tokenId);
-        burnAmount[_collectionID] = burnAmount[_collectionID] + 1;
+        burnAmount[_collectionID] += 1;
     }
 
     // burn to mint called from minterContract
@@ -213,12 +213,12 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
         require(msg.sender == minterContract, "Caller is not the Minter Contract");
         require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
-        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
+        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply += 1;
         if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
             _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
             // burn token
             _burn(_tokenId);
-            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
+            burnAmount[_burnCollectionID] += 1;
         }
     }
 

```

[NextGenCore#L180](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L180)

[NextGenCore#L191](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L191)

[NextGenCore#L195](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L195)

[NextGenCore#L197](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L197)

[NextGenCore#L208](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L208)

[NextGenCore#L216](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L216)

[NextGenCore#L221](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L221)

> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `NextGenMinterContract.airDropTokens` | 744940 | 744913 | -27 | -0.00% |
| `NextGenMinterContract.mint` | 404474 | 404463 | -11 | -0.00% |
| `NextGenCore` | 5501759 | 5489461 | -12298 | -0.22% |
            

## **G-7** | Possible Optimizations in `mint` function 


```diff
diff --git a/MinterContract.sol b/aMinterContract.sol
index df6f38d..83964a2 100644
--- a/MinterContract.sol
+++ b/aMinterContract.sol
@@ -194,15 +194,15 @@ contract NextGenMinterContract is Ownable {
     // mint function
 
     function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
-        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
+        require(setMintingCosts[_collectionID], "Set Minting Costs");
         uint256 col = _collectionID;
         address mintingAddress;
         uint256 phase;
         string memory tokData = _tokenData;
         if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
             phase = 1;
-            bytes32 node;
-            if (_delegator != 0x0000000000000000000000000000000000000000) {
+            bytes32 node; 
+            if (_delegator != address(0)) {
                 bool isAllowedToMint;
                 isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
                 if (isAllowedToMint == false) {
@@ -231,10 +231,14 @@ contract NextGenMinterContract is Ownable {
         collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
         require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
-        for(uint256 i = 0; i < _numberOfTokens; i++) {
+        uint256 i;
+        do{
             uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
             gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
-        }
+            unchecked {
+                ++i;
+            }
+        }while(i<_numberOfTokens);
         collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
         // control mechanism for sale option 3
         if (collectionPhases[col].salesOption == 3) {

```
[MinterContract#L196-L254](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L196-L254)
> Gas diff

| Method call or Contract deployment | Before | After | After - Before | (After - Before) / Before |
| :- | :-: | :-: | :-: | :-: |
| `NextGenMinterContract.mint` | 404474 | 404327 | -147 | -0.04% |
| `NextGenMinterContract` | 5454331 | 5450182 | -4149 | -0.08% |

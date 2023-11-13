## Low - 01 Not necessary to store two variables in storage with the same value
In NextGenCore.sol, `struct collectionAdditonalDataStructure` is defined with two elements both storing the same `randomizerContract` address. And in the contract, both values are modified every time randomizerContract is updated.

instance(1):

```solidity
// smart-contracts/NextGenCore.sol
struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
        uint256 reservedMinTokensIndex;
        uint256 reservedMaxTokensIndex;
        uint setFinalSupplyTimeAfterMint;
        //@audit-info ! No need to store both the address version and the contract version of the same data.
|>      address randomizerContract;
|>      IRandomizer randomizer;
    }
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L52-L53)
```solidity
// smart-contracts/NextGenCore.sol
function addRandomizer(
        uint256 _collectionID,
        address _randomizerContract
    ) public FunctionAdminRequired(this.addRandomizer.selector) {
        require(
            IRandomizer(_randomizerContract).isRandomizerContract() == true,
            "Contract is not Randomizer"
        );
        //@audit only one of these two variables needs to be declared and updated.
|>      collectionAdditionalData[_collectionID]
            .randomizerContract = _randomizerContract;
|>      collectionAdditionalData[_collectionID].randomizer = IRandomizer(
            _randomizerContract
        );
    }
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L172-L173)

Recommendations:
Only declare and update one of the two variables that hold the randomizer contract address.

## Low - 02 Not necessary to maintain an additional mapping from `_mintIndex` -> `_collectionID`
In NextGenCore.sol, a mapping `tokenIdsToCollectionIds` is written to every time a token is minted. 
This is unnecessary and can be simplified. Because minting is always called from MinterContract.sol and the token index is always calculated based collection id and total circulation of that colleciton id as follows:` uint256 mintIndex = gencore.viewTokensIndexMin(col) +
                gencore.viewCirSupply(col);` , and minimum index is always defined as `            collectionAdditionalData[_collectionID]
                .reservedMinTokensIndex = (_collectionID * 10000000000);`
, we always know a collection id given a mintIndex by (mintIndex/10000000000). Instead of a simple binary operation, keeping a separate mapping is more expensive.

instance(1):

```solidity
// smart-contracts/NextGenCore.sol
function _mintProcessing(
        uint256 _mintIndex,
        address _recipient,
        string memory _tokenData,
        uint256 _collectionID,
        uint256 _saltfun_o
    ) internal {
...
// @audit this mapping is written every time for a mint, the declaration and maintaining of this mapping is not necessary and can be simplified
|>      tokenIdsToCollectionIds[_mintIndex] = _collectionID;
        _safeMint(_recipient, _mintIndex);
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L230)

Recommendations:
Use a helper function with a simple binary operation to return collectionId for a mintIndex when needed, instead of maintaining mintIndex->collectionId mapping.

## Low - 03 NextGenCore.sol's supportsInterface() is incorrectly implemented
In NextGenCore.sol, it inherits ERC721Enumerable and ERC2981. However, calling supportsInterface() on NextGenCore will not show that it supports ERC2981 due to an incorrect override.

This is due to linearized inheritance order has ERC721Enumerable listed before ERC2981, therefore,super will refer to ERC721Enumerable's supportsInterface() implementation. The linearized inheritance order determines which parent contract super refers to when there are multiple inherited functions.

Contract inheritance: `contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {`

instance(1):

```solidity
// smart-contracts/NextGenCore.sol
    //@audit this will not shown supporting ERC2981 even though it supports ERC2981, due to the order of inheritance of NextGenCore. 
    function supportsInterface(
        bytes4 interfaceId
    ) public view virtual override(ERC721Enumerable, ERC2981) returns (bool) {
|>      return super.supportsInterface(interfaceId);
    }
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L338)

ERC721Enumerable:
```solidity
    function supportsInterface(bytes4 interfaceId) public view virtual override(IERC165, ERC721) returns (bool) {
        return interfaceId == type(IERC721Enumerable).interfaceId || super.supportsInterface(interfaceId);
    }
```
ERC2981:
```solidity
    function supportsInterface(bytes4 interfaceId) public view virtual override(IERC165, ERC165) returns (bool) {
        return interfaceId == type(IERC2981).interfaceId || super.supportsInterface(interfaceId);
    }
```
Recommendations:
In NexGenCore.sol - supportsInterface(), explicit return the interfaces needed to return.

## Low - 04 `tokenURI()` might return incomplete or incorrect tokenURI without warning
In NextGenCore.sol, `tokenURI()` includes multiple metadata of a token including `retrieveGenerativeScript(tokenId)` which concatenates a random token Hash generated by the randomizer contract. Due to the possible off-chain implementation (e.g. Chainlink VRF), token hash may not be generated in time or fail to generate. In this case, token Hash will be empty and `tokenURI()` will return an incorrect tokenURI. 

instance(1):

```solidity
// smart-contracts/NextGenCore.sol
    function tokenURI(
        uint256 tokenId
    ) public view virtual override returns (string memory) {
...
            string memory b64 = Base64.encode(
                abi.encodePacked(
                    '<html><head></head><body><script src="',
                    collectionInfo[tokenIdsToCollectionIds[tokenId]]
                        .collectionLibrary,
                    '"></script><script>',
// @audit retrieveGenerativeScript will return incorrect string when token hash is empty
|>                  retrieveGenerativeScript(tokenId),
                    "</script></body></html>"
                )
            );
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L353)
```solidity
// smart-contracts/NextGenCore.sol
    function retrieveGenerativeScript(
        uint256 tokenId
    ) public view returns (string memory) {
...
        return
            string(
                abi.encodePacked(
                    "let hash='",
// @audit when token hash is empty (not yet returned by randomizer contract or void due to error), tokenToHash[tokenId] will simply return bytes32(0), however, this will not be checked in tokenURI.
// @audit incorrect script string and tokenURI will be returned without handling
|>                  Strings.toHexString(uint256(tokenToHash[tokenId]), 32),
                    "';let tokenId=",
                    tokenId.toString(),
                    ";let tokenData=[",
                    tokenData[tokenId],
                    "];",
                    scripttext
                )
            );
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L456)

Recommendations:
When random token hash for a given tokenID is not available, either revert or return an empty string as warning, instead of returning an incorrect tokenURI with no warning.

## Low - 05 Unused Ownable (Note: Not included in the bot report)
There are (4) instances where the contract inherits Ownable.sol, however, owner is not used in any ways, making the inheritance redundant.

Instances(4):

```solidity
// smart-contracts/NextGenCore.sol
//@audit Ownable is not necessary based on current implementation, no onlyOwner function is used, nor is the variable `owner` accessed by any contracts.
contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L22)
```solidity
// smart-contracts/MinterContract.sol
//@audit Ownable is not necessary based on current implementation, no onlyOwner function is used, nor is the variable `owner` accessed by any contracts.
contract NextGenMinterContract is Ownable {
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L22)

```solidity
// smart-contracts/RandomizerVRF.sol

contract NextGenRandomizerVRF is VRFConsumerBaseV2, Ownable {
...

```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L19)

```solidity
// smart-contracts/RandomizerRNG.sol

contract NextGenRandomizerRNG is ArrngConsumer, Ownable {
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L18C1-L18C58)

Recommendations:
Consider not inheriting Ownable.sol when the owner is not used. 

## Low - 06 Verify the length of multiple dynamic arrays matches before for-loop (Note: Not included the bot report)

In MinterContract.sol - airDropTokens(), multiple tokens can be minted for multiple recipients each with different data or number of tokens. Although this is an admin funciton, there should be check to ensure various array lengths match each other to prevent errors.

instance(1):

```solidity
// smart-contracts/MinterContract.sol
//@audit Verify the length of the _recipients matches, the lenght of _collectionID and _numberOfTokens
    function airDropTokens(
        address[] memory _recipients,
        string[] memory _tokenData,
        uint256[] memory _saltfun_o,
        uint256 _collectionID,
        uint256[] memory _numberOfTokens
    ) public FunctionAdminRequired(this.airDropTokens.selector) {
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181)

Recommendations:
Check array lengths match each other.

## Low - 07 Unnecessary code, two require statements to ensure the same condition
In MinterContract.sol - `mint()`, there are two require statements both ensure that a collection maximum allowance is not exceeded is minted. This is unnecessary, only one require statement is needed.

instance(1):

```solidity
// smart-contracts/MinterContract.sol
    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
...
//@audit if the second require statement passes, the first one will always pass. So only the second require statement is needed.
            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L223-L224)

## Low - 08 Unnecessary code: `collectionTokenMintIndex` already calculated the new index Id, no need to declare new variable `mintIndex` and perform the same math again
In MinterContract.sol, there are (3) instances where `mintIndex` is declared and the same math is performed the second time when `collectionTokenMintIndex` could have been directly used. This is unnecessary code implementation.

Instances(3):

```solidity
// smart-contracts/MinterContract.sol
    function burnToMint(
        uint256 _burnCollectionID,
        uint256 _tokenId,
        uint256 _mintCollectionID,
        uint256 _saltfun_o
    ) public payable {
...
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex =
            gencore.viewTokensIndexMin(_mintCollectionID) +
            gencore.viewCirSupply(_mintCollectionID);
        require(
            collectionTokenMintIndex <=
                gencore.viewTokensIndexMax(_mintCollectionID),
            "No supply"
        );
...
// @audit `collecntionTokenMintIndex` can be used here without `mintIndex`
        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) +
            gencore.viewCirSupply(_mintCollectionID);
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L267)
```solidity
// smart-contracts/MinterContract.sol
    function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
...
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
// @audit `collecntionTokenMintIndex` can be used here without `mintIndex`
|>      uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L281)

Recommendations:
In the cases listed, `collectionTokenMintIndex` can be used directly since only 1 token is minted.

## Low - 09 Redundant code: `collectionArtistPrimaryAddress` status is already checked to be false at the beginning of the function. (Note: Not included in the bot report)
In MinterContract.sol, there are (2) instances where `collectionArtistPrimaryAddress[_collectionID].status` is rewritten with the same value. These are not included in the bot report.

instances(2):

```solidity 
// smart-contracts/MinterContract.sol
    function proposePrimaryAddressesAndPercentages(
...
        require(
            collectionArtistPrimaryAddresses[_collectionID].status == false,
            "Already approved"
        );
...
// @audit No need for this assignment, since it was already checked to be false;(Note: not included in the bot report)
       collectionArtistPrimaryAddresses[_collectionID].status = false;
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L389)
```solidity
// smart-contracts/MinterContract.sol
function proposeSecondaryAddressesAndPercentages(
...
        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
...
// @audit No need for this assignment, since it was already checked to be false;(Note: not included in the bot report)
      collectionArtistSecondaryAddresses[_collectionID].status = false;
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L403C3-L403C74)

Recommendations:
No need for the assignment here.

## Low - 10 `getWord()` has incorrect implementation and can be simplified
In XDandoms.sol - `getWord()`, an if statement is used to check whether id==0. However, this check also results in both when `id==0` and when `id==1` return the same word. 

In addition, the `if` statement can be removed to simply return `string[i]` and since `string[100]` is fixed at 100 elements, the input id will between [0,99] to fetch a word.

```solidity
// smart-contracts/XRandoms.sol
    function getWord(uint256 id) private pure returns (string memory) {
...
        //@audit (1) this different id 0 or 1 will return the same word; (2) for array retrieval, this can be simplified to string[i]
        // returns a word based on index
        if (id == 0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
}
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L28-L31)

Recommendations:
To avoid two ids returning the same word, simply use `string[i]` and ensure input is between [0,99].

## Low - 11 Unnecessary storage declaration. (Note: Not included in bot reports)
There are (2) instances where `tokenIdCollection` is declared and maintained but it's not necessary, since the info can be queried from NextGenCore contract getter.

instances(2):

```solidity
// smart-contracts/RandomizerRNG.sol
//@audit: tokenIdToColleciton can be queried from core contract getter. no need to declare and write to a new storage mapping
    mapping(uint256 => uint256) public tokenIdToCollection;
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L27)

```solidity
// smart-contracts/RandomizerVRF.sol
//@audit Low: tokenIdToColleciton can be queried from the core contract getter. no need to declare and write to a new storage mapping
    mapping(uint256 => uint256) public tokenIdToCollection;
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L31)

Recommendations:
Use the getter from NextGenCore.sol instead of maintaining a separate mapping that is supposed to contain the same data.

## Low - 12 Unnecessary if statement in AuctionDemo
In AuctionDemo.sol - `returnHighestBid()`, a local `highBid` and `index` are declared to be assigned in a for-loop. After the for-loop, there is an `if` statement where `auctionInfoData[_tokenid][index].status == true` is checked but this is unnecessary since this is already checked in the for-loop. 

In addition, instead simply return `highBid`. Because if the `highBid` is found in the for-loop, `highBid` will be returned and if `highBid` is not found, `highBid` will remain default `0` and returned. This saves the need of `if` statement bypass and simplify the process.

instances(1):

```solidity
// smart-contracts/AuctionDemo.sol
    function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
...
            //@audit unecessary if statement, because (1) there is valid bid in the for-loop, which will set highBid to a non-zero value, and reset index,
            //@audit .status will have already been checked to be ture (2) if no valid bid in the for-loop, .status will be false, highbid will remain default 0.
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L75-L78)

## Low - 13 Risk of dust bidding or an overwhelming number of users bidding in AuctionDemo.sol DOS `claimAuction()`
In AuctionDemo.sol, dust bidding can be passed in `participateToAuction()`. When enough dust bidding is passed. It might permanently DOS `claimAuction()` where for-loop is iterating over every single bid and send refunds to each bidder. 

Because `claimAuction()` is more gas intensive compared to `participateToAuction()`, when dust bidding is allowed. It's possible that `claimAuction()` will be DOSsed for the collection. Funds can be stuck in the AucitonDemo contract since there are no methods to recover funds from the owner.

The impact is Medium impact since funds lock and nft transfer can be DOSed. Due to the fact that current `participateToAuction()` will cause user gas. It will be expansive and rewardless for this vulnerability to be exploited. However, it's still possible for enough users to participate in the auction which overwhelms the contract resulting in `claimAuction()` DOSsed.

instances(1):

```solidity
// smart-contracts/AuctionDemo.sol
//@audit dust bidding is possible, to increase the auctionInfoData[_tokenid] to a large size, which might potentially create DOS for claimAuction();
//@audit Also, an overwhelming number of biddings can also DOS claimAuction();

    function participateToAuction(uint256 _tokenid) public payable {
|>      require(
            msg.value > returnHighestBid(_tokenid) &&
                block.timestamp <= minter.getAuctionEndTime(_tokenid) &&
                minter.getAuctionStatus(_tokenid) == true
        );
        auctionInfoStru memory newBid = auctionInfoStru(
            msg.sender,
            msg.value,
            true
        );
        auctionInfoData[_tokenid].push(newBid);
    }
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L58) 

Recommendations:
(1) Consider setting a minimal bidding cost variable for each collection to prevent large amount of dust bidding, raising the bidding threshold to an appropriate level;
(2) Consider separate claimAuction() logics into two or more functions to allow nft transfer, highest bid funds transfer, and other bidders refunds to be done in separate transactions to avoid DOS;

## Low - 14 `viewMaxAllowance(col)` can be easily bypassed by user who mint through `burnOrSwapExternalToMint()` or `burnToMint()`
In MinterContract.sol - `mint()`, `viewMaxAlloance(col)` is checked to ensure a single user account cannot mint more than allowed for a collection. This is a cap for a single-user account per collection id.

However, this check can be bypassed if the collection id has either `burnToMint()` or `burnOrSwapToMint()` enabled where `viewMaxAlloance(col)` is not checked in those user flows. This is a backdoor minting to bypass `viewMaxAlloance(col)`.

instances(2):

```solidity
// smart-contracts/MinterContract.sol
    function mint(
        uint256 _collectionID,
        uint256 _numberOfTokens,
        uint256 _maxAllowance,
        string memory _tokenData,
        address _mintTo,
        bytes32[] calldata merkleProof,
        address _delegator,
        uint256 _saltfun_o
    ) public payable {
...
// @audit this check ensures that for a single user account, in the public minting phase, one cannot pass the cap `gencore.viewMaxAllowance(col)`. However, this can be bypassed in `burnToMint()` or `burOrSwapToMint()`
            require(
                gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) +
                    _numberOfTokens <=
                    gencore.viewMaxAllowance(col),
                "Max"
            );
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L224)
```solidity
//  smart-contracts/MinterContract.sol   
function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
...
// @audit only total supply is checked here, no check on personal cap `gencore.viewMaxAllowance(col)`
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L264-L266)

Recommendation:
Enforce the same check on `gencore.viewMaxAllowance(col)` cap in `burnToMint()` and `burnOrSwapExternalToMint()`.

## NC - 01 `registerFuncitonAdmin()` will not distinguish two contracts with same funciton selectors, collision of admin might happen which might erode the access control.
In NextGenAdmins.sol - `registerFunctionAdmin()`, only function selector is used as key to assign function admin address, the contract name or address is not used. When there are two functions that have different contracts have the same function selector, this will create a collision of the admin value if the function admin should be different.

instance(1):

```solidity
// smart-contracts/MinterContract.sol
    function registerFunctionAdmin(
        address _address,
        bytes4 _selector,
        bool _status
    ) public AdminRequired {
// @audit this doesn't distinguish the same function selector but from two different contracts
        functionAdmin[_address][_selector] = _status;
    }
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L45)

Recommendations:
Consider hashing _selector and contact name for the key to avoid potential collision

## NC - 02 Unused imports Ownable.sol (Note: Not included in bot report)
There is (1) instance where the import statement of Ownable.sol is used but Ownable is not inherited or used at all.

instance(1)

```solidity
// smart-contracts/RandomizerNXT.sol
//@audit  Unused import ownable.sol (Note: Not included in bot report)
import "./Ownable.sol";
...
//@audit contract doesn't inherit Ownable 
contract NextGenRandomizerNXT {
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L15)

Recommendation:
Remove the import statement.

## NC - 03 Constructor doesn't need visibility declaration
In AuctionDemo.sol, the constructor is declared with a public keyword, this is unnecessary.

instance(1):

```solidity
// smart-contracts/AuctionDemo.sol
    constructor(
        address _minter,
        address _gencore,
        address _adminsContract
|>  ) public {
...
```
(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L36)



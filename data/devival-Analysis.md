The scope covered: all files in scope besides RandomizerVRF.sol.
Time spent on the codebase: 57.7 hours

Analysis of the codebase as a whole and any observations or advice they have about architecture, mechanism, or approach:
Having spent almost 60 hours reviewing the NextGen codebase, the code quality can be graded at most 5/10. 
The major issues besides High/Medium/Low that require attention from the team include:
Collections' and tokens' data is distributed among different contracts and numerous mappings, which makes it hard to understand, takes longer to audit, and, as a result has an increased risk of having bugs. It is strongly suggested for the team to create a separate Storage.sol file to keep all the data in one place.
Many variables/functions/mappings have grammar and vocabulary mistakes and need to be renamed.
The code does not follow Solidity Style Guides in many instances
Some functions are explained neither in docs nor in code comments. For instance: NextGenCore.sol such functions as airDropTokens(), mint(), burn(), burnToMint()
Furthermore, in [Sales Models → Periodic Sale Example](https://seize-io.gitbook.io/nextgen/for-creators/sales-models#sales-models-examples) has a wrong graph where after 5 NFTs minted, the price equals 1.4 ETH, however, according to the price formula `collectionMintCost + (collectionMintCost / rate ) * circulatingSupply` in getPrice function, the result has to be `1 + (1 / 10) * 5 = 1.5 ETH` [MinterContract.sol#L535-L539](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L535-L539) 
Many functions are not written according to Style Guides and do not follow the Check-Effects-Interactions pattern.
The codebase has plenty of redundant code, for instance, assigning variables for parameters twice inside the function. For instance:
```solidity
    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
 >>       uint256 col = _collectionID;
 >>       address mintingAddress;
 >>       uint256 phase;
 >>       string memory tokData = _tokenData;
```
[MinterContract.sol#L198-L201](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L198-L201)

or `payArtist` function:
```solidity
    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
        uint256 royalties = collectionTotalAmount[_collectionID];
        collectionTotalAmount[_collectionID] = 0;
        address tm1 = _team1;
        address tm2 = _team2;
        uint256 colId = _collectionID;

```
[MinterContract.sol#L415-L423](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L415-L423)
The line coverage percentage is not 100%, as the description claims, the average coverage is 60% and AuctionDemo.sol has 0%.
The codebase is heavily centralized due to the admins' ability to modify most of the variables.
The whole review process was done manually with VSCode and Hardhat. Bottom to top approach was taken, in this case first where checked Randomizer, then the Admins contract, AuctionDemo, and then the main ones, such as Minter and Core contracts.
Having found 8 Medium and 2 High severity issues it is strongly suggested for the NextGen team to apply for another audit after fixing the current issues.

P.S. Tried formatting by <details></details> tag did not seem to work in preview. All work was donemanually, no bots. Just focused to put as much time as possible work full-time on this, to find every single instance that could be changed to make the codebase both secure and high-quality.

### Time spent:
60 hours
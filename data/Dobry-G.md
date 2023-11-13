# Some variables should be immutable
#### Instances: 5
For example, variables that are only set in the constructor and never edited after that, consider marking them as immutable, as it would avoid the expensive storage-writing operations.


#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol
26: IMinterContract public minter;
27: INextGenAdmins public adminsContract;
28: address gencore;

#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

119: IDelegationManagementContract private dmc;

#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol

105: address public minterContract;


# Use named variables in function returns
#### Instances: 1
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol
35: function randomNumber() public view returns (uint256){
In the () brackets it should be uint256 randomNumber, and then the variable randomNumber will be returned automatically without the need of a return statement in the function body, which will lead to a significant gas cost savings.

# Public functions not called by the contract should be declared external
This leads to gas savings
#### Instances: 26
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol

Function should be external instead of public because it is not called in the contract.
53: function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {


#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol
Function should be external instead of public because it is not called in the contract.
71: function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {


#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol

Function should be external instead of public because it is not called in the contract.
38: function registerAdmin(address _admin, bool _status) public onlyOwner {
44: function registerFunctionAdmin(address _address, bytes4 _selector, bool _status) public AdminRequired {
50: function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
58: function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
65: function retrieveGlobalAdmin(address _address) public view returns(bool) {
71: function retrieveFunctionAdmin(address _address, bytes4 _selector) public view returns(bool) {
77: function retrieveCollectionAdmin(address _address, uint256 _collectionID) public view returns(bool) {

#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

326: function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {

470: function retrievePrimarySplits(uint256 _collectionID) public view returns(uint256, uint256){

476: function retrievePrimaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){

482: function retrieveSecondarySplits(uint256 _collectionID) public view returns(uint256, uint256){

488: function retrieveSecondaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){

494: function retrieveCollectionPhases(uint256 _collectionID) public view returns(uint, uint, bytes32, uint, uint){

500: function retrieveCollectionMintingDetails(uint256 _collectionID) public view returns(uint256, uint256, uint256, uint256, uint8, address){

#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

257: function artistSignature(uint256 _collectionID, string memory _signature) public {


343: function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {

367: function collectionFreezeStatus(uint256 _collectionID) public view returns(bool){

415: function retrieveTokensAirdroppedPerAddress(uint256 _collectionID, address _address) public view returns(uint256) {

426: function retrieveCollectionInfo(uint256 _collectionID) public view returns(string memory, string memory, string memory, string memory, string memory, string memory){

432: function retrieveCollectionLibraryAndScript(uint256 _collectionID) public view returns(string memory, string[] memory){

438: function retrieveCollectionLibraryAndScript(uint256 _collectionID) public view returns(string memory, string[] memory){

444: function retrieveTokenHash(uint256 _tokenid) public view returns(bytes32){

461: function totalSupplyOfCollection(uint256 _collectionID) public view returns (uint256) {

467: function retrievetokenImageAndAttributes(uint256 _tokenId) public view returns(string memory, string memory) {

# Struct packing
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

#### Instances: 6

We can use the uint types uint8, uint16, uint32, etc, though Solidity reserves 256 bits of storage regardless of the uint type meaning, in normal instances, there's no cost saving. However, if you have multiple uints inside a struct, using a smaller-sized uint when possible will allow Solidity to pack these variables together to take up less storage.

44: struct collectionPhasesDataStructure {
63: struct royaltiesPrimarySplits {
73: struct collectionPrimaryAddresses {
88: struct collectionPrimaryAddresses {
98: struct collectionSecondaryAddresses {


#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

44: struct collectionAdditonalDataStructure {


# Instead of accessing data on the blockchain multiple times, use a local variable in the function and access it’s value
Each reading from a data stored on the blockchain costs resources and it would be more efficient if the reading is from a local variable

#### Instances: 1

#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

530: function getPrice(uint256 _collectionId) public view returns (uint256) {
532: if (collectionPhases[_collectionId].salesOption == 3) {
535: if (collectionPhases[_collectionId].rate > 0) {
536: return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
540: } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){
546: tDiff = (block.timestamp - collectionPhases[_collectionId].allowlistStartTime) / collectionPhases[_collectionId].timePeriod;
549: if (collectionPhases[_collectionId].rate == 0) {
550: price = collectionPhases[_collectionId].collectionMintCost / (tDiff + 1);
551: decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
553: if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
554: price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);
556: price = collectionPhases[_collectionId].collectionEndMintCost;
559: if (price - decreaserate > collectionPhases[_collectionId].collectionEndMintCost) {
562: return collectionPhases[_collectionId].collectionEndMintCost;
566: return collectionPhases[_collectionId].collectionMintCost;

# Don't compare boolean expressions to boolean literals
#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol

#### Instances: 36

Instead of
if(x == true)
use
if(x)

instead of (x == false)
use
if(!x)

117: require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

124: require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

148: require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");

158: } else if (artistSigned[_collectionID] == false) {

171: require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");

239: require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

259: require(artistSigned[_collectionID] == false, "Already Signed");

267: require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

274: require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "Data frozen");

283: require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");

293: require(isCollectionCreated[_collectionID] == true, "No Col");

316: require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");

323: require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");

345: if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {

348: } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {

#### https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

137: require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

144: require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

151: require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

158: require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

171: require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

182: require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

197: require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

208: if (isAllowedToMint == false) {

211: require(isAllowedToMint == true, "No delegation");

259: require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");

277: require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

309: require((gencore.retrievewereDataAdded(_burnCollectionID) == true) && (gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

317: require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

328: require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");

329: require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");

334: if (isAllowedToMint == false) {

337: require(isAllowedToMint == true, "No delegation");

381: require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");

395: require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");

416: require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");

455: require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
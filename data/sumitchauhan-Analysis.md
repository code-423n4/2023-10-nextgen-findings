1. Use assembly to write address storage values:
  NextGenCore.sol => require(newOwner != address(0), "Ownable: new owner is the zero address");
  NextGenCore.sol => require(owner != address(0), "ERC721: address zero is not a valid owner");
  NextGenCore.sol => require(owner != address(0), "ERC721: invalid token ID");
  NextGenCore.sol => return _ownerOf(tokenId) != address(0);
  NextGenCore.sol => require(to != address(0), "ERC721: mint to the zero address");
  NextGenCore.sol => require(to != address(0), "ERC721: transfer to the zero address");
  NextGenCore.sol => if (from == address(0)) {
  NextGenCore.sol => if (to == address(0)) {
  NextGenCore.sol => if (royalty.receiver == address(0)) {
  NextGenCore.sol => if (receiver == address(0)) {
  NextGenCore.sol => if (receiver == address(0)) {
  MinterContract.sol => require(newOwner != address(0), "Ownable: new owner is the zero address");
  RandomizerNXT.sol => require(newOwner != address(0), "Ownable: new owner is the zero address");
 
2. Using storage instead of memory for structs/arrays saves gas:
  NextGenCore.sol => string memory buffer = new string(length);
  NextGenCore.sol => bytes memory buffer = new bytes(2 * length + 2);
  NextGenCore.sol => string memory table = _TABLE;
  NextGenCore.sol => string memory result = new string(4 * ((data.length + 2) / 3));
  NextGenCore.sol => string memory baseURI = _baseURI();
  NextGenCore.sol => RoyaltyInfo memory royalty = _tokenRoyaltyInfo[tokenId];
  NextGenCore.sol => string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
  NextGenCore.sol => string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
  MinterContract.sol => bytes32[] memory hashes = new bytes32[](totalHashes);
  MinterContract.sol => bytes32[] memory hashes = new bytes32[](totalHashes);
  MinterContract.sol => string memory tokData = _tokenData;
  MinterContract.sol => string memory tokData = _tokenData;


3. Use calldata instead of memory for function parameters:
  NextGenCore.sol => function functionCall(address target, bytes memory data) internal returns (bytes memory) {
  NextGenCore.sol => function functionStaticCall(address target, bytes memory data) internal view returns (bytes memory) {
  NextGenCore.sol => function functionDelegateCall(address target, bytes memory data) internal returns (bytes memory) {
  NextGenCore.sol => function _revert(bytes memory returndata, string memory errorMessage) private pure {
  NextGenCore.sol => function equal(string memory a, string memory b) internal pure returns (bool) {
  NextGenCore.sol => function encode(bytes memory data) internal pure returns (string memory) {
  NextGenCore.sol => function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data) public virtual override {
  NextGenCore.sol => function _safeTransfer(address from, address to, uint256 tokenId, bytes memory data) internal virtual {
  NextGenCore.sol => function _safeMint(address to, uint256 tokenId, bytes memory data) internal virtual {
  NextGenCore.sol => function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) public FunctionAdminRequired(this.createCollection.selector) {
  NextGenCore.sol => function updateCollectionInfo(uint256 _collectionID, string memory _newCollectionName, string memory _newCollectionArtist, string memory _newCollectionDescription, string memory _newCollectionWebsite, string memory _newCollectionLicense, string memory _newCollectionBaseURI, string memory _newCollectionLibrary, uint256 _index, string[] memory _newCollectionScript) public CollectionAdminRequired(_collectionID, this.updateCollectionInfo.selector) {
  NextGenCore.sol => function artistSignature(uint256 _collectionID, string memory _signature) public {
  NextGenCore.sol => function changeTokenData(uint256 _tokenId, string memory newData) public FunctionAdminRequired(this.changeTokenData.selector) {
  NextGenCore.sol => function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
  MinterContract.sol => function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {


4. Reduce the size of error messages:
  NextGenCore.sol => require(success, "Address: unable to send value, recipient may have reverted");
  NextGenCore.sol => require(owner != address(0), "ERC721: address zero is not a valid owner");
  NextGenCore.sol => require(_isApprovedOrOwner(_msgSender(), tokenId), "ERC721: caller is not token owner or approved");
  NextGenCore.sol => require(_isApprovedOrOwner(_msgSender(), tokenId), "ERC721: caller is not token owner or approved");
  NextGenCore.sol => require(_checkOnERC721Received(from, to, tokenId, data), "ERC721: transfer to non ERC721Receiver implementer");
  NextGenCore.sol => require(index < ERC721.balanceOf(owner), "ERC721Enumerable: owner index out of bounds");
  NextGenCore.sol => require(index < ERC721Enumerable.totalSupply(), "ERC721Enumerable: global index out of bounds");
  NextGenCore.sol => require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");
  NextGenCore.sol => require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");

5. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables:
  NextGenCore.sol => result += 1;
  NextGenCore.sol => result += 128;
  NextGenCore.sol => result += 64;
  NextGenCore.sol => result += 32;
  NextGenCore.sol => result += 16;
  NextGenCore.sol => result += 8;
  NextGenCore.sol => result += 4;
  NextGenCore.sol => result += 2;
  NextGenCore.sol => result += 1;
  NextGenCore.sol => result += 64;
  NextGenCore.sol => result += 32;
  NextGenCore.sol => result += 16;
  NextGenCore.sol => result += 8;
  NextGenCore.sol => result += 4;
  NextGenCore.sol => result += 2;
  NextGenCore.sol => result += 1;
  NextGenCore.sol => result += 16;
  NextGenCore.sol => result += 8;
  NextGenCore.sol => result += 4;
  NextGenCore.sol => result += 2;
  NextGenCore.sol => result += 1;
  NextGenCore.sol => _balances[to] += 1;
  NextGenCore.sol => _balances[owner] -= 1;
  NextGenCore.sol => _balances[from] -= 1;
  NextGenCore.sol => _balances[to] += 1;
  NextGenCore.sol => _balances[account] += amount;



6. Constants in comparisons should appear on the left side:
  NextGenCore.sol => if (returndata.length == 0) {
  NextGenCore.sol => if (returndata.length > 0) {
  NextGenCore.sol => if (prod1 == 0) {
  NextGenCore.sol => if (rounding == Rounding.Up && mulmod(x, y, denominator) > 0) {
  NextGenCore.sol => if (a == 0) {
  NextGenCore.sol => if (value >> 128 > 0) {
  NextGenCore.sol => if (value >> 64 > 0) {
  NextGenCore.sol => if (value >> 32 > 0) {
  NextGenCore.sol => if (value >> 16 > 0) {
  NextGenCore.sol => if (value >> 8 > 0) {
  NextGenCore.sol => if (value >> 4 > 0) {
  NextGenCore.sol => if (value >> 2 > 0) {
  NextGenCore.sol => if (value >> 1 > 0) {
  NextGenCore.sol => if (value >= 10 ** 64) {
  NextGenCore.sol => if (value >= 10 ** 32) {
  NextGenCore.sol => if (value >= 10 ** 16) {
  NextGenCore.sol => if (value >= 10 ** 8) {
  NextGenCore.sol => if (value >= 10 ** 4) {
  NextGenCore.sol => if (value >= 10 ** 2) {
  NextGenCore.sol => if (value >= 10 ** 1) {
  NextGenCore.sol => if (value >> 128 > 0) {
  NextGenCore.sol => if (value >> 64 > 0) {
  NextGenCore.sol => if (value >> 32 > 0) {
  NextGenCore.sol => if (value >> 16 > 0) {
  NextGenCore.sol => if (value >> 8 > 0) {
  NextGenCore.sol => if (value == 0) break;
  NextGenCore.sol => require(value == 0, "Strings: hex length insufficient");
  NextGenCore.sol => if (data.length == 0) return "";
  NextGenCore.sol => require(newOwner != address(0), "Ownable: new owner is the zero address");
  NextGenCore.sol => require(owner != address(0), "ERC721: address zero is not a valid owner");
  NextGenCore.sol => require(owner != address(0), "ERC721: invalid token ID");
  NextGenCore.sol => require(to != address(0), "ERC721: mint to the zero address");
  NextGenCore.sol => require(to != address(0), "ERC721: transfer to the zero address");
  NextGenCore.sol => if (reason.length == 0) {
  NextGenCore.sol => if (batchSize > 1) {
  NextGenCore.sol => if (from == address(0)) {
  NextGenCore.sol => if (to == address(0)) {
  NextGenCore.sol => if (royalty.receiver == address(0)) {
  NextGenCore.sol => if (receiver == address(0)) {
  NextGenCore.sol => if (receiver == address(0)) {
  NextGenCore.sol => require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
  NextGenCore.sol => if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
  NextGenCore.sol => if (phase == 1) {
  NextGenCore.sol => if (_index == 1000) {
  NextGenCore.sol => } else if (_index == 999) {
  NextGenCore.sol => require(tokenToHash[_mintIndex] == 0x0000000000000000000000000000000000000000000000000000000000000000);
  NextGenCore.sol => if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
  NextGenCore.sol => } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
  MinterContract.sol => if (totalHashes > 0) {
  MinterContract.sol => } else if (leavesLen > 0) {
  
 
7. Unsafe casting may overflow:
  NextGenCore.sol => return x + (int256(uint256(x) >> 255) & (a ^ b));
  NextGenCore.sol => return uint256(n >= 0 ? n : -n);
  NextGenCore.sol => return toHexString(uint256(uint160(addr)), _ADDRESS_LENGTH);
  NextGenCore.sol => return string(abi.encodePacked("let hash='",Strings.toHexString(uint256(tokenToHash[tokenId]), 32),"';let tokenId=",tokenId.toString(),";let tokenData=[",tokenData[tokenId],"];", scripttext));

 
8. Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`):
  NextGenCore.sol => if (value >= 10 ** 64) {
  NextGenCore.sol => value /= 10 ** 64;
  NextGenCore.sol => if (value >= 10 ** 32) {
  NextGenCore.sol => value /= 10 ** 32;
  NextGenCore.sol => if (value >= 10 ** 16) {
  NextGenCore.sol => value /= 10 ** 16;
  NextGenCore.sol => if (value >= 10 ** 8) {
  NextGenCore.sol => value /= 10 ** 8;
  NextGenCore.sol => if (value >= 10 ** 4) {
  NextGenCore.sol => value /= 10 ** 4;
  NextGenCore.sol => if (value >= 10 ** 2) {
  NextGenCore.sol => value /= 10 ** 2;
  NextGenCore.sol => if (value >= 10 ** 1) {
  NextGenCore.sol => return result + (rounding == Rounding.Up && 10 ** result < value ? 1 : 0);
  


### Time spent:
18 hours
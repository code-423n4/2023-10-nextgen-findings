## [G‑01]Consider using ERC721A instead of ERC721

```
 constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L108C4-L108C106

```
  require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L205C7-L205C110

```
require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L215C9-L215C104

```
  address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L108C7-L108C67

```
  IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112C15-L112C90

## [G-02] Use hardcode address instead address(this)
Instead of using address(this) , it is more gas-efficient to pre-calculate and use the hardcoded
address . Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

```
uint balance = address(this).balance;
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L462C9-L462C46

```
   uint balance = address(this).balance;
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L80C6-L80C46

## [G-03] Make for loop unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is
limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time
and gas to let the for loop overflow.

```
   for (uint256 x; x < _tokenId.length; x++) {
            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
            _requireMinted(_tokenId[x]);
            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
        }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L282C6-L287C10

```
 for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
            scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i])); 
        }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L453C8-L455C10

```
for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L184C9-L185C146

```
 for (uint256 i=0; i<_selector.length; i++) {
            functionAdmin[_address][_selector[i]] = _status;
        }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L51C8-L53C10

```
  for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L69C11-L73C18

```
  for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L90C7-L93C14

```
  for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L110C7-L114C75

```
   for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L136C6-L138C61

## [G-04] Events are not indexed
The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to
filter the events efficiently.
Recommend adding the indexed keyword in each event,
```
 event RequestFulfilled(uint256 requestId, uint256[] randomWords);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L20C4-L20C70

## [G-05] Using private rather than public for constants, saves gas
If needed, the value can be read from the verified contract source code. Savings are due to the
compiler not having to create non-payable getter functions for deployment calldata, and not adding
another entry to the method ID table.

```
 uint32 public callbackGasLimit = 40000;
  uint16 public requestConfirmations = 3;
    uint32 public numWords = 1;

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L27C4-L30C1


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Claiming auction and get money back | 1 |
| [L-2](#L-2) | DoS due to uncheck of `allowlistEndTime` and `allowlistStartTime` | 1 |
### [L-1] Claiming auction and get money back
If for any reason the auctionDemo has received ETH, then the auction winner can both claim his auction by calling the the `claimAuction()` and after that in the same block he can call `cancelBid`
This is due to cancelBid not checking if the auction was already claimed.

#### PoC
```solidity
    function cancelBid(uint256 _tokenid, uint256 index) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
        auctionInfoData[_tokenid][index].status = false;
        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
    }
```
[Github](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L124-L130)

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
        uint256 highestBid = returnHighestBid(_tokenid);
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
        address highestBidder = returnHighestBidder(_tokenid);
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```
[Github](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L104-L120)


Also this action is possible due to the following check:

```solidity
require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
```
#### Recommandation
Check if the auction is still not claimed, otherwise don't accept canceling the bid as funds would be already refunded

### [L-2] DoS due to uncheck of `allowlistEndTime`/`allowlistStartTime` and `publicStartTime`/`publicEndTime`
When a collection phases are set using the `MinterContract.setCollectionPhases()` the value of `allowlistEndTime` and `allowlistStartTime` are not checked. In addition, `publicStartTime` and `publicEndTime` are not checked. 
Therefore, if for any reason a value of `allowlistEndTime < allowlistStartTime` and `publicEndTime`<`publicStartTime` is introduced, the minting functions will be broken and we will not be able to mint any token.


#### PoC
```solidity
    function setCollectionPhases(uint256 _collectionID, uint _allowlistStartTime, uint _allowlistEndTime, uint _publicStartTime, uint _publicEndTime, bytes32 _merkleRoot) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
        collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
        collectionPhases[_collectionID].merkleRoot = _merkleRoot;
        collectionPhases[_collectionID].publicStartTime = _publicStartTime;
        collectionPhases[_collectionID].publicEndTime = _publicEndTime;
    }
```
[Github](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L170-L177)

```solidity
    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        uint256 col = _collectionID;
        address mintingAddress;
        uint256 phase;
        string memory tokData = _tokenData;
        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
            phase = 1;
            bytes32 node;
            if (_delegator != 0x0000000000000000000000000000000000000000) {
                bool isAllowedToMint;
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
                if (isAllowedToMint == false) {
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);    
                }
                require(isAllowedToMint == true, "No delegation");
                node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));
                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");
                mintingAddress = _delegator;
            } else {
                node = keccak256(abi.encodePacked(msg.sender, _maxAllowance, tokData));
                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, msg.sender) + _numberOfTokens, "AL limit");
                mintingAddress = msg.sender;
            }
            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');
        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
            phase = 2;
            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
            mintingAddress = msg.sender;
            tokData = '"public"';
        } else {
            revert("No minting");
        }
		// ...
```
[Github](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L196-L229)


#### Recommandation
in the setCollectionPhases() function add the following checks:
- `allowlistStartTime < allowlistEndTime`
- `publicEndTime < publicStartTime`



## Non-critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Some functions in `auctionDemo` contract need tests | 1 |
| [NC-2](#NC-2) | Unusual counter in `for` loop | 1 |

### [NC-1] Some functions in `auctionDemo` contract need tests
Not all the `auctionDemo` contract functions has test code in nextGen.test.js file. A better code coverage means less bug and vulnerabilities.


### [NC-2] Unusual counter in `for` loop
Try to use common variable names for `for`-loops for better readability


```solidity

//@audit `y` variable name is unusual
for (uint256 y=0; y< _recipients.length; y++) {

```

[Github](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L184)
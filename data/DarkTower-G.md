## Summary

### Gas Optimizations
| |Issue|Contexts|
|-|:-|:-|
| GAS-01 | Unnecessary state updates in `proposePrimaryAddressesAndPercentage` function  | 1 |
| GAS-02 | `mintIndex` can be calculated once and used across the function lifecycle | 4 | 
| GAS-03 | `tokensMintedAllowlistAddress`, `tokensMintedPerAddress`, `tokensAirdropPerAddress` mappings can be combined into one mapping to reduce gas cost for the protocol  | 1 |
| GAS-04 | Redundant checks are implemented in several functions which are not required, taking extra gas | 3 |

Total: 9 contexts over 4 issues

## Gas Optimizations
### GAS-01: Unnecessary state updates in `proposePrimaryAddressesAndPercentage` function
`collectionArtistPrimaryAddresses[_collectionID].status` is already checked to be `false` when the function runs thereby attempting to change the state to `false` again is unnecessary.

Context: MinterContract::proposePrimaryAddressesAndPercentage
```solidity
function proposePrimaryAddressesAndPercentages(uint256 _collectionID, address _primaryAdd1, address _primaryAdd2, address _primaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {
 @>     require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd1 = _primaryAdd1;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd2 = _primaryAdd2;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd3 = _primaryAdd3;
        collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
@>      collectionArtistPrimaryAddresses[_collectionID].status = false;
    }

```
Context: MinterContract::proposeSecondaryAddressesAndPercentage
```solidity
    function proposeSecondaryAddressesAndPercentages(uint256 _collectionID, address _secondaryAdd1, address _secondaryAdd2, address _secondaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposeSecondaryAddressesAndPercentages.selector) {
@>        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1 = _secondaryAdd1;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2 = _secondaryAdd2;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3 = _secondaryAdd3;
        collectionArtistSecondaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add3Percentage = _add3Percentage;
@>      collectionArtistSecondaryAddresses[_collectionID].status = false;
    }
```


### GAS-02: `mintIndex` can be calculated once and used across the function lifecycle
`mintIndex` is derived from `gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID)`, which is then used in the function. Keep in mind `collectionTokenMintIndex` does the exact same thing and is already declared now left unused. Sticking to just one of these two will save on gas.

Context: MinterContract::airDropTokens
```solidity
189   uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```
Context: MinterContract::mint
```solidity
237    uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
```
Context: MinterContract::burnToMint
```solidity
270    uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);

```
Context: MinterContract::mintAndAuction
```solidity
284    uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```
### GAS-03: `tokensMintedAllowlistAddress`, `tokensMintedPerAddress`, `tokensAirdropPerAddress` mappings can be combined into one mapping to reduce gas cost for the protocol
The protocol's declaration of mappings such as `tokensMintedAllowlistAddress`, `tokensMintedPerAddress`, `tokensAirdropPerAddress`, can be combined to save gas.

Context: NextGenCore
```solidity
    mapping (uint256 => mapping (address => uint256)) private tokensMintedPerAddress;

    // minted tokens per address per collection during allowlist
    mapping (uint256 => mapping (address => uint256)) private tokensMintedAllowlistAddress;

    // tokens airdrop per address per collection 
    mapping (uint256 => mapping (address => uint256)) private tokensAirdropPerAddress;
```

### GAS-04: Redundant checks are implemented in several functions which are not required, taking extra gas
Redundant checks in these contexts are unnecessary as these conditions are already true. 

Context: AuctionDemo::returnHighestBid
```solidity
function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
@>          if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```

Context: AuctionDemo::returnHighestBidder
```solidity
    function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
        }
@>      if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }
```

Context: NextGenCore::airDropTokens
```solidity
    function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
@>      if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }
```

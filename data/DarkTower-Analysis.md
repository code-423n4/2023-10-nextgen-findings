# NextGen Protocol Analysis Report

## Introduction
This analysis report goes over the various components and sections of the NextGen Protocol as it has been audited by the DarkTower team. During the duration of this audit, we aimed primarily to gain a comprehensive understanding of the codebase and try to figure out issues that may become vulnerable to the protocol, it's functioning and ultimately affect its users.

#### Findings Summary
Severity  | Instances 
------------- | -------------
Medium  | 6
High | 1
Low | 3
Gas | 4

## Comments for judge to contextualize the findings
This is to help the judge reduce the time spent on judging. Our findings are mainly Loss of funds, dos, re-entrancy etc. that protocol team need to fix before going live. Judges don't need to confirm with the sponsors for the validity of the issues we raised as we have have provided all the required attack scenarios/proof-of-concepts along with our findings. For this audit, we are also submitting low/gas or non critical issues.

## Approach taken in evaluating the codebase
* We began the audit on the day of its audit inception directly going over the protocol's documentation to get an idea of components and architecture of the protocol.

* We moved to codebase first looking into NextGen Core contract and then process went on. We have confirmed the functionality and understand the contracts well enough and with our auditing skills, have analyzed the contracts with different attack possibilities.

* We proceeded to audit the market components and found some medium and high vulnerabilities the protocol should look into fixing with our suggestions or other best practices they put together in that regard. 

* We conducted the tests provided by the protocol team and tried to figure out some serious edge cases covered and not yet covered with the tests.

* Finally we wrote the reports and submitted to the c4 portal.

## Codebase overview
NextGen is a NFT protocol which purpose is to deal with  generative AI art and other non-art NFTs use cases in the blockchain. NextGen allows to create multiple ERC712 collections under a single on-chain address. There are 4 main on chain contracts: `Core`, `Minter`, `Admin` and `Randomizer`.The core contract interacts with the other three for its functionality.

## Project Uniqueness
Having wide range of minting models with different phases functionality makes the project unique to a certain extent. Various sales models of the protocol are also unique to other protocols.

## Architecture recommendations

#### Current Architecture
There are four main contracts `Core`, `Minter`, `Admin` and `Randomizer` in the protocol:
1. `Core`: It is the contract where the ERC721 tokens are minted and includes all the core functions of the ERC721 standard. The core contracts is where other contracts interact with. The core contracts holds the details of a collection of NFTs. 

2. `Minter`: This contract interact with the core to mint an ERC712 token for a collection. The minter contract holds the information about primary, secondary addresses, merkle roots, sales models  etc.
3. `Admin`: The `admin` contract acts as a main contract for adding and removing of global admins, function level admins, collection level admin etc.

4. `Randomizer`: There are 3 main randomizer contracts: `RandomizerVRF`, `RandomizerRNG`, `RandomizerNXT` for generating random has for each token during minting process. 

`AuctionDemo` is another important contract that was in scope, this contract implements the auction method for users to bid for assets in the protocol.

The complexity management of the protocol is not very to up the required standard. The centralization risks are there as admins can call certain functions that can steal or rug pull user assets. Having low complexity management in the current architecture makes more attack space for hackers to exploit the protocol.

#### Recommendations
1. Most of the low level calls in the protocol doesn't check for its success boolean to confirm the transactions. We recommend adding checks after a low level call.
Example: In MinterContract, 
```solidity
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```

2. The storage mappings and variables can be moved to one single contract so that risk of making mistake using them. We recommend adding these mappings, variables, constants in one contract.

4. Many important functions lacks event emissions which can reduce off-chain interactions. We recommend adding events in functions. Example: MinterContract::setCollectionCosts
``` solidity
    function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 
                _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address 
                _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
 		        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
 		        collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
 		        collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
 		        collectionPhases[_collectionID].rate = _rate;
 		        collectionPhases[_collectionID].timePeriod = _timePeriod;
 		        collectionPhases[_collectionID].salesOption = _salesOption;
 		        collectionPhases[_collectionID].delAddress = _delAddress;
 		        setMintingCosts[_collectionID] = true;
 		    }  
```

5. The auction contract could be heavily improved in terms of refunds. Changing from the current state of refunds in the function `claimAuction` to a Pull over push pattern where every bidder can get a refund at any time from the functions `cancelBid` && `cancelAllBids` (except for the winner if he claims the reward), can significantly reduce the errors in the contract.

6. In the auction contract consider improving the logic of the function `returnHighestBidder`. As of now the `highBid` is not getting updated and the function always compares `auctionInfoData[_tokenid][i].bid` to 0. This works for now as the mapping `mapping (uint256 => auctionInfoStru[]) public auctionInfoData` is sorted, but can lead to many issues in the future if logic changes.


## Codebase Quality Analysis
Overall, the quality of the codebase is good. The covered tests are looking good. 
Category  | Comments | Results
------------- | ------------- | -------------
Code Comments  | Not NatSpc followed, some comments are missing |Good
Documentation | Functions in components could be explained a little bit more in comments and doc | Better
Complexity Management    | Protocol doesn't handle the complexity of the codebase properly | Good
Other test coverage    | Most of the functional tests are covered with Foundry  | Excellent


## Centralization Risks
A malicious function admin can reset the MinterContract balance to zero anytime they choose thereby disrupting usual accounting an flow of funds. Keep in mind since this contract holds the funds from minting for all collections, an admin calling it every time will end up wiring the contract's balance to the owner but that will cause tedious work when reimbursing each individual artist of a collection for example as the collection could easily be over a hundred to thousands of collections' funds wired now needing reimbursement and temporarily pausing crucial functions such as payArtist if the contract has no balance.
 
## System Risks 
1. As protocol uses onchain randomizer such as Chainlink. There are some of the ways that miners or validators can potentially manipulate randomness generation. [Link](https://docs.chain.link/vrf/v2/security#overview) to Chainlink VRF security considerations.
2. Smart contracts can have vulnerabilities related to its logic, access errors that can be exploited by the attackers. We recommend adding fixes that are being identified by the warden of Code4rena.
3. Due to external dependencies in some functions such as `mint`, read only reentrancy or price manipulations can be possible.
#### Time Spent
Around 10 days.
96 Hours.























### Time spent:
96 hours
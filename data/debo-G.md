## [G-01] DEFINE CONSTRUCTOR AS PAYABLE
**Impact**
Developers can save around 10 opcodes and some gas if the constructors are defined as payable.
However, it should be noted that it comes with risks because payable constructors can accept ETH during deployment.
**POC**
Here is a proof of concept (POC) for defining constructors as payable in Solidity:
```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract PayableConstructor {
    uint256 public value;

    // Payable constructor
    constructor() payable {
        // The value variable is set to the amount of wei sent with the contract deployment
        value = msg.value;
    }
}
```
In this POC, the constructor is defined as payable, which means it can accept ETH during the deployment of the contract. The value variable is set to the amount of wei sent with the contract deployment (msg.value).

To deploy this contract while sending ETH, you would use something like this in your deployment script (assuming you're using Truffle):
```js
const PayableConstructor = artifacts.require("PayableConstructor");

module.exports = function(deployer) {
  // Deploy the contract with 10 ETH
  deployer.deploy(PayableConstructor, {value: web3.utils.toWei("10", "ether")});
};
```
While this can save some gas because it avoids the need for a separate function to accept ETH, it comes with risks. If the contract deployment fails for any reason, the sent ETH could be lost. Therefore, it's recommended to only use payable constructors when you have a specific reason to do so and understand the risks involved.
**Remediation**
It is suggested to mark the constructors as payable to save some gas. 
Make sure it does not lead to any adverse effects in case an upgrade pattern is involved.
**Locations**
```txt
smart-contracts/AuctionDemo.sol#L36-L40
smart-contracts/RandomizerNXT.sol#L25-L30
smart-contracts/NextGenAdmins.sol#L26-L28
smart-contracts/RandomizerRNG.sol#L29-L33
smart-contracts/RandomizerVRF.sol#L39-L45
smart-contracts/NextGenCore.sol#L108-L112
smart-contracts/MinterContract.sol#L129-L133
```
## [G-02] CHEAPER INEQUALITIES IN REQUIRE()
**Impact**
The contract was found to be performing comparisons using inequalities inside the require statement. 
When inside the require statements, non-strict inequalities (>=, <=) are usually costlier than strict equalities (>, <).
**POC**
Here is a proof of concept (POC) for using strict inequalities inside the require statement in Solidity:
```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract InequalityComparison {
    uint256 public value;

    // Function that uses non-strict inequality in require statement
    function setValueNonStrict(uint256 _value) public {
        // This require statement uses a non-strict inequality
        require(_value >= 0, "Value must be non-negative");
        value = _value;
    }

    // Function that uses strict inequality in require statement
    function setValueStrict(uint256 _value) public {
        // This require statement uses a strict inequality
        require(_value > 0, "Value must be positive");
        value = _value;
    }
}
```
In this POC, the setValueNonStrict function uses a non-strict inequality (>=) in its require statement, while the setValueStrict function uses a strict inequality (>).

While the difference in gas costs between these two types of inequalities is usually small, using strict inequalities can be slightly more efficient. However, the choice between non-strict and strict inequalities should primarily be based on the logic of your contract, not gas optimisation.
**Remediation**
It is recommended to go through the code logic, and, if possible, modify the non-strict inequalities with the strict ones to save ~3 gas as long as the logic of the code is not affected.
**Locations**
```txt
smart-contracts/AuctionDemo.sol#L58-L58
smart-contracts/AuctionDemo.sol#L105-L105
smart-contracts/AuctionDemo.sol#L125-L125
smart-contracts/AuctionDemo.sol#L135-L135
smart-contracts/NextGenCore.sol#L148-L148
smart-contracts/NextGenCore.sol#L206-L206
smart-contracts/MinterContract.sol#L186-L186
smart-contracts/MinterContract.sol#L213-L213
smart-contracts/MinterContract.sol#L217-L217
smart-contracts/MinterContract.sol#L223-L223
smart-contracts/MinterContract.sol#L224-L224
smart-contracts/MinterContract.sol#L232-L232
smart-contracts/MinterContract.sol#L233-L233
smart-contracts/MinterContract.sol#L251-L251
smart-contracts/MinterContract.sol#L260-L260
smart-contracts/MinterContract.sol#L261-L261
smart-contracts/MinterContract.sol#L265-L265
smart-contracts/MinterContract.sol#L266-L266
smart-contracts/MinterContract.sol#L280-L280
smart-contracts/MinterContract.sol#L294-L294
smart-contracts/MinterContract.sol#L339-L339
smart-contracts/MinterContract.sol#L360-L360
smart-contracts/MinterContract.sol#L361-L361
```
## [G-03] ARRAY LENGTH CACHING
**Impact**
During each iteration of the loop, reading the length of the array uses more gas than is necessary. 
In the most favourable scenario, in which the length is read from a memory variable, storing the array length in the stack can save about 3 gas per iteration. 
In the least favourable scenario, in which external calls are made during each iteration, the amount of gas wasted can be significant.
**POC**
Here is a proof of concept (POC) for optimising gas usage by storing the array length in a stack variable instead of reading it from memory during each iteration of a loop:
```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract ArrayLengthOptimization {
    uint256[] public array;

    // Function that reads array length from memory during each iteration
    function sumArrayNonOptimized() public view returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < array.length; i++) {
            sum += array[i];
        }
        return sum;
    }

    // Function that stores array length in a stack variable
    function sumArrayOptimized() public view returns (uint256) {
        uint256 sum = 0;
        uint256 length = array.length;
        for (uint256 i = 0; i < length; i++) {
            sum += array[i];
        }
        return sum;
    }
}
```
In this POC, the sumArrayNonOptimized function reads the array length from memory during each iteration of the loop, while the sumArrayOptimized function stores the array length in a stack variable before the loop starts.
The optimized function can save about 3 gas per iteration, which can add up to significant savings for large arrays. However, the actual savings will depend on the specific details of your contract and the current gas price.
**Locations**
`smart-contracts/AuctionDemo.sol#L90-L94`
The following array was detected to be used inside loop without caching it's value in memory: auctionInfoData.
`smart-contracts/AuctionDemo.sol#L110-L119`
The following array was detected to be used inside loop without caching it's value in memory: auctionInfoData.
`smart-contracts/AuctionDemo.sol#L136-L142`
The following array was detected to be used inside loop without caching it's value in memory: auctionInfoData.
`smart-contracts/NextGenAdmins.sol#L51-L53`
The following array was detected to be used inside loop without caching it's value in memory: _selector.
`smart-contracts/NextGenCore.sol#L282-L287`
The following array was detected to be used inside loop without caching it's value in memory: _tokenId.
`smart-contracts/NextGenCore.sol#L453-L455`
The following array was detected to be used inside loop without caching it's value in memory: scripttext.
`smart-contracts/MinterContract.sol#L184-L191`
The following array was detected to be used inside loop without caching it's value in memory: _recipients.
**Remediation**
Consider storing the array length of the variable before the loop and use the stored length instead of fetching it in each iteration.
## [G-04] CHEAPER INEQUALITIES IN IF()
**Impact**
The contract was found to be doing comparisons using inequalities inside the if statement.
When inside the if statements, non-strict inequalities (>=, <=) are usually cheaper than the strict equalities (>, <).
**POC**
The contract uses strict inequalities in several places, such as in the participateToAuction function where it checks if the sent value is greater than the highest bid, and in the returnHighestBid function where it checks if a bid is greater than the current highest bid.
Here is a proof of concept (POC) that demonstrates how changing these to non-strict inequalities could save gas:
```sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.19;

import "./IMinterContract.sol";
import "./IERC721.sol";
import "./Ownable.sol";
import "./INextGenAdmins.sol";

contract auctionDemo is Ownable {

    // ...

    // participate to auction
    function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value >= returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }

    // get highest bid
    function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid >= highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }

    // ...

}
```
In this POC, the strict inequalities in the participateToAuction and returnHighestBid functions have been changed to non-strict inequalities. 
This should result in slightly lower gas costs when these functions are called.
**Remediation**
It is recommended to go through the code logic, and, if possible, modify the strict inequalities with the non-strict ones to save ~3 gas as long as the logic of the code is not affected.
**Location**
```txt
smart-contracts/AuctionDemo.sol#L67-L67
```
## [G-05] OPTIMIZING ADDRESS ID MAPPING
**Impact**
Combining multiple address/ID mappings into a single mapping using a struct enhances storage efficiency, simplifies code, and reduces gas costs, resulting in a more streamlined and cost-effective smart contract design.
It saves storage slot for the mapping and depending on the circumstances and sizes of types, it can avoid a Gsset (20000 gas) per mapping combined. 
Reads and subsequent writes can also be cheaper when a function requires both values and they fit in the same storage slot.
**POC**
Here is a proof of concept (POC) for combining multiple mappings into a single mapping using a struct in 
Solidity:
```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract CombinedMapping {
    // Define a struct to hold multiple values
    struct Data {
        uint256 id;
        address addr;
    }

    // Single mapping that uses the struct
    mapping (uint256 => Data) public dataMap;

    // Function to set values in the combined mapping
    function setData(uint256 key, uint256 _id, address _addr) public {
        Data storage data = dataMap[key];
        data.id = _id;
        data.addr = _addr;
    }

    // Function to get values from the combined mapping
    function getData(uint256 key) public view returns (uint256, address) {
        Data storage data = dataMap[key];
        return (data.id, data.addr);
    }
}
```
In this POC, the CombinedMapping contract uses a struct (Data) to hold multiple values (id and addr) in a single mapping (dataMap). 
The setData function sets values in the combined mapping, and the getData function retrieves values from the combined mapping.
This design is more storage-efficient and cost-effective than using separate mappings for each value. 
It saves a storage slot for each mapping combined, which can avoid a 20000 gas cost (SSTORE opcode) per mapping. 
Reads and subsequent writes can also be cheaper when a function requires both values and they fit in the same storage slot.
**Locations**
```txt
smart-contracts/NextGenCore.sol#L74-L74
smart-contracts/NextGenCore.sol#L77-L77
smart-contracts/NextGenCore.sol#L80-L80
```
**Remediation**
It is suggested to modify the code so that multiple mappings using the address->id parameter are combined into a struct.
## [G-06] STORAGE VARIABLE CACHING IN MEMORY
**POC**
The `AuctionDemo.sol` contract uses the minter state variable multiple times in the participateToAuction function. 
This results in multiple SLOAD operations, which are more expensive than MLOAD/MSTORE operations.
Here is a proof of concept (POC) that demonstrates how storing the minter state variable in memory could save gas:
```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.19;

import "./IMinterContract.sol";
import "./IERC721.sol";
import "./Ownable.sol";
import "./INextGenAdmins.sol";

contract auctionDemo is Ownable {

    // ...

    // participate to auction
    function participateToAuction(uint256 _tokenid) public payable {
        IMinterContract memory minterMemory = minter;
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minterMemory.getAuctionEndTime(_tokenid) && minterMemory.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }

    // ...

}
```
In this POC, the minter state variable is stored in memory at the start of the participateToAuction function. This memory variable is then used in the require statement, reducing the number of SLOAD operations and potentially saving gas.
`smart-contracts/AuctionDemo.sol#L26-L26`
**Impact**
The contract auctionDemo is using the state variable minter multiple times in the function participateToAuction.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/AuctionDemo.sol#L50-L50`
The contract auctionDemo is using the state variable auctionInfoData multiple times in the function returnHighestBid.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/AuctionDemo.sol#L50-L50`
The contract auctionDemo is using the state variable auctionInfoData multiple times in the function returnHighestBidder.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/AuctionDemo.sol#L26-L26`
The contract auctionDemo is using the state variable minter multiple times in the function claimAuction.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/AuctionDemo.sol#L28-L28`
The contract auctionDemo is using the state variable gencore multiple times in the function claimAuction.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/AuctionDemo.sol#L50-L50`
The contract auctionDemo is using the state variable auctionInfoData multiple times in the function claimAuction.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/AuctionDemo.sol#L53-L53`
The contract auctionDemo is using the state variable auctionClaim multiple times in the function claimAuction.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/AuctionDemo.sol#L50-L50`
The contract auctionDemo is using the state variable auctionInfoData multiple times in the function cancelBid.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/RandomizerNXT.sol#L20-L20`
The contract NextGenRandomizerNXT is using the state variable randoms multiple times in the function calculateTokenHash.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/RandomizerRNG.sol#L20-L20`
The contract NextGenRandomizerRNG is using the state variable requestToToken multiple times in the function fulfillRandomWords.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/RandomizerVRF.sol#L33-L33`
The contract NextGenRandomizerVRF is using the state variable requestToToken multiple times in the function fulfillRandomWords.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L26-L26`
The contract NextGenCore is using the state variable newCollectionIndex multiple times in the function .
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L26-L26`
The contract NextGenCore is using the state variable newCollectionIndex multiple times in the function createCollection.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L41-L41`
The contract NextGenCore is using the state variable collectionInfo multiple times in the function createCollection.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L57-L57`
The contract NextGenCore is using the state variable collectionAdditionalData multiple times in the functions setCollectionData; addRandomizer; airDropTokens;
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L80-L80`
The contract NextGenCore is using the state variable tokensAirdropPerAddress multiple times in the function airDropTokens.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L57-L57`
The contract NextGenCore is using the state variable collectionAdditionalData multiple times in the function mint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L74-L74`
The contract NextGenCore is using the state variable tokensMintedPerAddress multiple times in the function mint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L77-L77`
The contract NextGenCore is using the state variable tokensMintedAllowlistAddress multiple times in the function mint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L57-L57`
The contract NextGenCore is using the state variable collectionAdditionalData multiple times in the function burn.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L83-L83`
The contract NextGenCore is using the state variable burnAmount multiple times in the function burn.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L57-L57`
The contract NextGenCore is using the state variable collectionAdditionalData multiple times in the function burnToMint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L68-L68`
The contract NextGenCore is using the state variable tokenIdsToCollectionIds multiple times in the function retrieveGenerativeScript.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/NextGenCore.sol#L95-L95`
The contract NextGenCore is using the state variable tokenImageAndAttributes multiple times in the function retrievetokenImageAndAttributes.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L59-L59`
The contract NextGenMinterContract is using the state variable collectionPhases multiple times in the function setCollectionCosts.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L118-L118`
The contract NextGenMinterContract is using the state variable gencore multiple times in the function airDropTokens.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L23-L23`
The contract NextGenMinterContract is using the state variable collectionTotalAmount multiple times in the function mint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L26-L26`
The contract NextGenMinterContract is using the state variable lastMintDate multiple times in the function mint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L119-L119`
The contract NextGenMinterContract is using the state variable dmc multiple times in the function mint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L26-L26`
The contract NextGenMinterContract is using the state variable lastMintDate multiple times in the function mintAndAuction.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L32-L32`
The contract NextGenMinterContract is using the state variable burnOrSwapIds multiple times in the function initializeExternalBurnOrSwap.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L23-L23`
The contract NextGenMinterContract is using the state variable collectionTotalAmount multiple times in the function burnOrSwapExternalToMint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L119-L119`
The contract NextGenMinterContract is using the state variable dmc multiple times in the function burnOrSwapExternalToMint.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L70-L70`
The contract NextGenMinterContract is using the state variable collectionRoyaltiesPrimarySplits multiple times in the function setPrimaryAndSecondarySplits.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L95-L95`
The contract NextGenMinterContract is using the state variable collectionRoyaltiesSecondarySplits multiple times in the function setPrimaryAndSecondarySplits.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L84-L84`
The contract NextGenMinterContract is using the state variable collectionArtistPrimaryAddresses multiple times in the function proposePrimaryAddressesAndPercentages.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L59-L59`
The contract NextGenMinterContract is using the state variable collectionPhases multiple times in the function retrieveCollectionPhases.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L59-L59`
The contract NextGenMinterContract is using the state variable collectionPhases multiple times in the function retrieveCollectionMintingDetails.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
`smart-contracts/MinterContract.sol#L59-L59`
The contract NextGenMinterContract is using the state variable collectionPhases multiple times in the function getPrice.
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
**Remediation**
Storage variables read multiple times inside a function should instead be cached in the memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.
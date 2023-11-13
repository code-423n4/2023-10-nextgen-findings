# QA Report

## Table of Contents

| Issue ID                                                                                                                 | Description                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [QA-01](#qa-01-all-queries-to-nextgenrandomizervrfrequestrandomwords-after-initial-deployment-of-contracts-would-revert) | All queries to `NextGenRandomizerVRF::requestRandomWords()` after initial deployment of contracts would revert                                 |
| [QA-02](#qa-02-minting-doesnt-work-as-intended-in-regards-to-the-accepted-limits-within-periods)                         | Minting doesn't work as intended in regards to the accepted limits within periods                                                              |
| [QA-03](#qa-03-emergencywithdraw-should-be-made-more-effective-insured-to-always-execute)                                | `emergencyWithdraw()` should be made more effective insured to always execute                                                                  |
| [QA-04](#qa-04-inconsistency-in-sister-function-implementations)                                                         | Inconsistency in sister function implementations                                                                                               |
| [QA-05](#qa-05-redundant-multplication-in-burnorswapexternaltomint-should-be-removed)                                    | Redundant multiplication in `burnOrSwapExternalToMint()` should be removed                                                                     |
| [QA-06](#qa-06-registerbatchfunctionadmin-takes-too-much-operational-overhead)                                           | `registerBatchFunctionAdmin()` takes too much operational overhead                                                                             |
| [QA-07](#qa-07-setters-should-always-have-equality-checkers)                                                             | Setters should always have equality checkers                                                                                                   |
| [QA-08](#qa-08-updateimagesandattributes-currently-employs-a-logical-downside)                                           | `updateImagesAndAttributes()` currently employs a logical downside                                                                             |
| [QA-09](#qa-09-redundant-validation-in-auction-bid-retrieval-should-be-removed)                                          | Redundant validation in auction bid retrieval should be removed                                                                                |
| [QA-10](#qa-10-updatecollectioninfo-should-be-made-more-effective)                                                       | `updateCollectionInfo()` should be made more effective                                                                                         |
| [QA-11](#qa-11-add-airdroptokens-documentation-to-the-docs)                                                              | Add `airDropTokens` documentation to the [NextGen Smart Contracts Core docs](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/core) |

## QA-01 All queries to `NextGenRandomizerVRF::requestRandomWords()` after initial deployment of contracts would revert

### Impact

Low, since this could easily be fixed even after project is live on the mainnet, but currently all queries to chainlink after initial deployment of contracts would never work due to the `keyHash` not been set with the hash for the `Ethereum mainnnet`

### Proof of Concept

The below has been stated in `RandomizerVRF.sol`

```solidity
    bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;
```

The above would cause requests to `Chainlink's` VRF to never work, cause from here: https://docs.chain.link/vrf/v2/subscription/examples/get-a-random-number

We can see that the `keyHash` is actually the gas lane that's to be used

```solidity
// The gas lane to use, which specifies the maximum gas price to bump to.
// For a list of available gas lanes on each network,
// see https://docs.chain.link/docs/vrf/v2/subscription/supported-networks/#configurations
    bytes32 keyHash =
        0x474e34a077df58807dbe9c96d3c009b23b3c6d0cce433e59bbf5b34f823bc56c;
```

Now, navigating to the supported gas lanes, one can see that this address `0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15` is actually meant for the Goerli testnet and not mainnet, as such all request to query this on mainnet would not work

ADD SCREENSHOT

### Recommended Mitigation Steps

Initialize the contract with a valid key hash from the start of the project

## QA-02 Minting doesn't work as intended in regards to the accepted limits within periods

### Impact

`mint()`is intended to be limited to minting one token per predefined time period, but this has a critical oversight. This flaw could potentially allow more frequent minting than intended, leading to unintended token supply inflation and a breach of the contract's designed tokenomics.

> NB: Submitting this as QA, cause it's borderline between `QA- "function incorrect as to spec(_i.e if current spec is incorrect_)" or 2-Med -> The function of the protocol or could be impacted`, leaving for judge to upgrade if they see fit.

### Proof of Concept

The concern revolves around the way `lastMintDate[col]` is updated in the function:

```solidity
function mint(uint256 _collectionID, ...) public payable {
    // ... other code ...

    if (collectionPhases[_collectionID].salesOption == 3) {
        uint timeOfLastMint;
        if (lastMintDate[_collectionID] == 0) {
            timeOfLastMint = collectionPhases[_collectionID].allowlistStartTime - collectionPhases[_collectionID].timePeriod;
        } else {
            timeOfLastMint = lastMintDate[_collectionID];
        }

        uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
        require(tDiff >= 1 && _numberOfTokens == 1, "1 mint/period");

        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
    }

    // ... other code ...
}
```

The issue arises when more than one `period` elapses without minting. For instance, if the period is set to one minute, and 10 minutes pass without minting, `lastMintDate[col]` would reflect a time 10 periods ago. The current implementation does not update `lastMintDate[col]` to the current timestamp, but instead, it relies on `viewCirSupply`, which can lead to multiple tokens being minted in the same period.

### Recommended Mitigation Steps

To rectify this, the `mint` function should be modified to update `lastMintDate[col]` with the current block timestamp right after a successful mint. This adjustment will ensure that `tDiff` is always calculated from the most recent minting event, effectively enforcing the "one mint per period" rule as per the contract's design. The revised update should be as follows:

```solidity
if (tDiff >= 1 && _numberOfTokens == 1) {
    lastMintDate[_collectionID] = block.timestamp; // Update with the current timestamp
    // Proceed with minting
}
```

Implementing this change will maintain the integrity of the minting frequency control, aligning the actual functionality of the contract with its intended design and documentation.

## QA-03 `emergencyWithdraw()` should be made more effective insured to always execute

> NB: This is a two in one report.

### Impact

#### 1

The emergency withdrawal function `emergencyWithdraw()` of the contract has hardcoded the receiver of the funds to the owner of the `adminsContract`. This design decision might expose the funds to permanent loss if the private key to the admin account is ever misplaced, stolen, or rendered inaccessible.

#### 2

The `emergencyWithdraw` function's primary purpose is to enable a quick and safe withdrawal of all contract funds in emergency scenarios.

### Proof of Concept

#### 1

Take a look at `emergencyWithdraw()`

```solidity
//@audit QA hardcoding the receiver of the funds
function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
    uint balance = address(this).balance;
    address admin = adminsContract.owner();
    (bool success, ) = payable(admin).call{value: balance}("");
    emit Withdraw(msg.sender, success, balance);
}
```

As observed, the recipient of the withdrawal is hardcoded to the `owner` of `adminsContract`. This design does not provide flexibility and could potentially lock funds if the admin account becomes inaccessible.

#### 2

This function or one in similar context is been used in multiple instances of code in scope, issue is that they all make an external call which is not in line with the concept of emergency withdrawals, as these external calls introduces new points of failures , and if influenced by the contract's owner or due to errors in the implementations, could cause the 'emergencyWithdraw' function to revert, compromising its primary feature.

From the implementation, it's evident there are potential pitfalls that could lead to the failure of the emergencyWithdraw function due to reasons such as:

- Owner can't receive ether
- Wrong address has been passed and stored as the `adminsContract` and we can't query `.owner()`

### Recommended Mitigation Steps

#### 1

Since there is already this protection : `FunctionAdminRequired(this.emergencyWithdraw.selector)` then we can just modify the `emergencyWithdraw()` function to accept the recipient's address as a parameter, allowing the calling admin to specify the withdrawal destination.

```diff
-   function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
+   function emergencyWithdraw(address payable _recipient) public FunctionAdminRequired(this.emergencyWithdraw.selector) {
    uint balance = address(this).balance;
-    address admin = adminsContract.owner();
-    (bool success, ) = payable(admin).call{value: balance}("");
+    (bool success, ) = _recipient.call{value: balance}("");
    emit Withdraw(msg.sender, success, balance);
}
```

#### 2

It is recommended to revise the `emergencyWithdraw` function to eliminate dependencies that might restrict its execution. For example, refer to this [Panckeswap Smart Contract](https://github.com/pancakeswap/pancake-smart-contracts/blob/d8f55093a43a7e8913f7730cfff3589a46f5c014/projects/smartchef/v2/contracts/SmartChefInitializable.sol#L235C5-L235C14) for a better approach.

## QA-04 Inconsistency in sister function implementations

### Impact

`NextGenRandomizerNXT.updateAdminsContract()` is potentially vulnerable to unintended or malicious updates to its admin contract. This is due to a missing check that ensures that the specified contract address actually pertains to an admin contract unlike in `NextGenRandomizerRNG.updateAdminsContract()`

### Proof of Concept

Take a look at `NextGenRandomizerNXT.updateAdminsContract()`

```solidity
//@audit QA Incosisitency in sister function implementation
function updateAdminsContract(address _admin) public FunctionAdminRequired(this.updateAdminsContract.selector) {
    adminsContract = INextGenAdmins(_admin);
}
```

Contrast this with the corresponding method in `NextGenRandomizerRNG`:

```solidity
function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
    require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
    adminsContract = INextGenAdmins(_newadminsContract);
}
```

As observed, the `NextGenRandomizerNXT` does not have the `require` check that validates if the given contract is indeed an admin contract, unlike its counterpart `NextGenRandomizerRNG`.

### Recommended Mitigation Steps

Add the same `require` check from `NextGenRandomizerRNG` to `NextGenRandomizerNXT` in the `updateAdminsContract` function.

## QA-05 Redundant multplication in `burnOrSwapExternalToMint()` should be removed

### Impact

Info, but presence of a redundant multiplication operation in the `burnOrSwapExternalToMint` function may lead to slightly increased gas costs and reduced code readability.

### Proof of Concept

In the `burnOrSwapExternalToMint` function, there is a line of code that checks if the ETH value sent (`msg.value`) is greater than or equal to the price required for the operation. However, the comparison includes a redundant multiplication by 1, which is unnecessary and does not alter the logic or outcome of the comparison. The specific line of code is:

```solidity
require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
```

This multiplication by 1 is extraneous and can be removed without affecting the functionality of the code. The revised line should simply be:

```solidity
require(msg.value >= getPrice(col), "Wrong ETH");
```

By removing the multiplication, the code becomes more concise and clear, potentially leading to minimal gas cost savings due to the reduced computational requirement.

### Recommended Mitigation Steps

Update the `require` statement to remove the multiplication by 1, simplifying the expression to directly compare `msg.value` with `getPrice(col)`.

## QA-06 `registerBatchFunctionAdmin()` takes too much operational overhead

### Impact

The current implementation of batch registration for function administrators in the NextGen Admin Contract easily leads to not just more stressed transactions but even increased transaction costs and operational overhead.

### Proof of Concept

Take a look at `registerBatchFunctionAdmin()`

```solidity
//@audit
function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
    for (uint256 i=0; i<_selector.length; i++) {
        functionAdmin[_address][_selector[i]] = _status;
    }
}
```

As seen, the function has an iterative approach used to set the status for each function selector. Although this approach is functional, it requires separate contract function calls if one needs to set some selectors as `true` and others as `false`. This approach can be optimized by accepting an array for statuses, which will allow a definition of different statuses for each selector within a single transaction.

### Recommended Mitigation Steps

Modify the `registerBatchFunctionAdmin` function to accept an array of bools (`bool[] memory _statuses`) instead of a single `bool _status`. This would allow individual status assignment for each selector in a single transaction.
If this would be done, then implement a check to ensure that the lengths of `_selector` and `_statuses` arrays are the same. This will prevent potential mismatches and errors.

## QA-07 Setters should always have equality checkers

### Impact

Low, info on how to have better code structure and ot going through unnecessary transactions.

### Proof of Concept

Take a look at `updateRNGCost()`

```solidity
    // function to update cost

    function updateRNGCost(uint256 _ethRequired) public FunctionAdminRequired(this.updateRNGCost.selector) {
        ethRequired = _ethRequired;
    }
```

As seen, the value of `ethRequired` is just set without any checks if it's the same value as previous which is not best practise.

### Recommended Mitigation Steps

Include equality checks in setter functions.

## QA-08 `updateImagesAndAttributes()` currently employs a logical downside

### Impact

In the current implementation of `updateImagesAndAttributes`, the function will revert if any one of the provided token IDs belongs to a frozen collection. This may lead to inefficiencies and inconvenience for the administrator when updating images and attributes for multiple tokens, especially if they have to cross-check and remove all token IDs related to frozen collections.

### Proof of Concept

Consider the function `updateImagesAndAttributes`:

```solidity
//@audit QA ...
function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public ... {
    for (uint256 x; x < _tokenId.length; x++) {
        require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
        ...
        tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
        tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
    }
}
```

The `require` statement within the loop ensures that the function will revert for any token ID within a frozen collection. So, if the admin wants to update attributes and images for 100 tokens, and just one of them is from a frozen collection, the entire transaction will fail.

### Recommended Mitigation Steps

Instead of using the `require` conditional statement that reverts the function, employ an `if` statement to skip the frozen collections and continue updating the rest, implementation could look like the below.

```solidity
   function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public ... {
       for (uint256 x; x < _tokenId.length; x++) {
           if (collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false) {
               _requireMinted(_tokenId[x]);
               tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
               tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
           }
       }
   }
```

## QA-09 Redundant validation in auction bid retrieval should be removed

### Impact

Low, info on how to get better code structure, cause presently the `returnHighestBid` function in `auctionDemo.sol` uses an iterative method to find the highest bid for a given token, even though the `participateToAuction` function ensures that every new bid is higher than the previous ones.

### Proof of Concept

Take a look at `participateToAuction()`

```solidity
    function participateToAuction(uint256 _tokenid) public payable {
      //@audit
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }
```

The above function ensures that any new bid is always higher than the previous ones.

However, the `returnHighestBid` function still employs a loop to check for the highest bid:

```solidity
//@audit there's no need for this `if (auctionInfoData[_tokenid][i].bid > highBid` as it would always be true
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
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```

Given that each new bid is higher than the last, the highest bid will always be the last item in the `auctionInfoData[_tokenid]` array, making the iterative search superfluous.

### Recommended Mitigation Steps

**Optimize the `returnHighestBid` function**: Instead of using a loop to find the highest bid, directly retrieve the last bid from the `auctionInfoData[_tokenid]` array, which is guaranteed to be the highest due to checks in `participateToAuction`.

```solidity
function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
    uint256 length = auctionInfoData[_tokenid].length;
    if (length > 0 && auctionInfoData[_tokenid][length - 1].status == true) {
        return auctionInfoData[_tokenid][length - 1].bid;
    }
    return 0;
}
```

## QA-10 `updateCollectionInfo()` should be made more effective

### Impact

Info since this heavily relies on admin mistake, but currently the `updateCollectionInfo` function contains a potential out-of-bounds vulnerability due to a lack of index range check when updating. If the provision of an index that is out of the valid range is made, it would result in unintended behavior, causing an out-of-bounds error.

### Proof of Concept

Take a look at `updateCollectionInfo()`

```solidity
    function updateCollectionInfo(uint256 _collectionID, string memory _newCollectionName, string memory _newCollectionArtist, string memory _newCollectionDescription, string memory _newCollectionWebsite, string memory _newCollectionLicense, string memory _newCollectionBaseURI, string memory _newCollectionLibrary, uint256 _index, string[] memory _newCollectionScript) public CollectionAdminRequired(_collectionID, this.updateCollectionInfo.selector) {
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
         if (_index == 1000) {
            collectionInfo[_collectionID].collectionName = _newCollectionName;
            collectionInfo[_collectionID].collectionArtist = _newCollectionArtist;
            collectionInfo[_collectionID].collectionDescription = _newCollectionDescription;
            collectionInfo[_collectionID].collectionWebsite = _newCollectionWebsite;
            collectionInfo[_collectionID].collectionLicense = _newCollectionLicense;
            collectionInfo[_collectionID].collectionLibrary = _newCollectionLibrary;
            collectionInfo[_collectionID].collectionScript = _newCollectionScript;
        } else if (_index == 999) {
            collectionInfo[_collectionID].collectionBaseURI = _newCollectionBaseURI;
        } else {
            //@audit
            collectionInfo[_collectionID].collectionScript[_index] = _newCollectionScript[0];
        }
    }
```

As stated in the comment: "If index is equal to a value that is within the indices range of the collectionScript the function updates that single part of the script.", yet there is no actual check to ensure the `_index` is within the valid range of `collectionScript`.

### Recommended Mitigation Steps

Before updating `collectionScript` with a specific `_index`, add a check to ensure that `_index` is within the valid range.

## QA-11 Add `airDropTokens` documentation to the [docs](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/core)

### Impact

[The documentation](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/core) is missing details about the `airDropTokens()` function. Without proper documentation, users and developers may find it difficult to understand its purpose and potential risks, leading to potential misuse, incorrect integrations, or security oversights.

### Proof of Concept

Referencing the `airDropTokens` function:

```solidity
    // airdrop called from minterContract
//@audit
    function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }

```

Note that a one liner explanation is obviously not enugh, also while other functions are explained, i.e their natspec is provided in the docs, but navigating to said [docs](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/core), it's evident that the documentation for this is missing unlike other typical functions (like `mint`, `burn`, etc.)

### Recommended Mitigation Steps

Update the official documentation to include a comprehensive section on the `airDropTokens` function. This section should cover:

- Purpose: Describe what the function is used for.
- Parameters: Explain each parameter's role and any constraints.
- Expected Behavior: Elaborate on the function's sequence of operations.
- Restrictions: Highlight the check for `msg.sender` being the minter contract.
- Use Cases: Provide typical scenarios where this function is beneficial.
- Possible Risks: Address any potential security or logical concerns.

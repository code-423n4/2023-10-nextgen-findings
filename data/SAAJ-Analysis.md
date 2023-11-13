
# Introduction
This analysis report is based on comprehensive review and analysis of NextGen protocol contract in scope for the C4 contest by focusing on ```AuctionDemo```, ```NextGenCore```, ```MinterContract```, ```NextGenAdmins```, ```RandomizerNXT```, ```RandomizerRNG```, ```RandomizerVRF``` and ```XRandoms```.

The analysis carries out assessment for all contracts in-scope for their functionalities, emphasizing potential vulnerabilities and areas for improvement. The approach i taken to carry out the audit is to identify various common security pitfalls that expands to several areas like code redundancy, access control, events, randomness, re-entrancy, integer over/underflow and effective means for error handling.

The review starts with analysing the mechanism review related to workflow of the code base. The second part highlights the approach taken to audit the project.  Afterwards the codebase quality is analyse which related to identify highlighting potential risks related to centralization and systemic, emphasizing the significant control held by specific roles or functions within the contracts, which could lead to issues with transparency and potential manipulation.

After identifying the risk, recommendations are made to enhance each contract's security and functionality like implementing more detailed event logging for state changes, refining access control mechanisms, revisiting the source of randomness used in contracts and potential code refinements to streamline and enhance the codebase.


# Mechanism Review
This section carryout the assessment of workflow for all contracts in scope for this audit. Each contract working mechanism is discussed as below:


- **AuctionDemo** 
```auctionDemo``` provides users to participate in the auction by calling the ```participateToAuction``` function with the token id and sending Ether. The function checks if the sent value is higher than the current highest bid, if the auction is still ongoing, and if the auction is active. If all conditions are met, a new bid is created and added to the auction data. The contract also provides functions to get the highest bid and the highest bidder for a given token id. These functions iterate over the auction data and return the highest bid and the corresponding bidder.

In the contract after the auction ends, the winner or an admin can claim the auction by calling the ```claimAuction``` function with the token id. The function checks if the auction has ended, if the auction has not been claimed yet, and if the auction is active. If all conditions are met, the token is transferred to the highest bidder, the highest bid is sent to the contract owner, and all other bidders are refunded. Bidders can cancel their bids before the auction ends by calling the ```cancelBid``` function with the token id and the index of their bid. The function checks if the auction is still ongoing and if the bidder is the sender of the transaction.

If all conditions are met, the bid is cancelled and the bidder is refunded. Bidders can also cancel all their bids before the auction ends by calling the ```cancelAllBids``` function with the token id. The function checks if the auction is still ongoing and cancels and refunds all bids from the sender.

- **MinterContract** 
```NextGenMinterContract``` contract has several functions related to minting new tokens. The mint function allows a user to mint new tokens in a specific collection. The ```burnToMint```  function allows a user to burn a token in one collection and mint a new one in another collection. It also maintains record through mappings for various collection-related data, such as the total amount collected during minting, timestamp of the last mint and burn or swap address for external collections. The contract maintains data related to royalties and artist addresses by keeping track of primary and secondary splits for royalties as well as primary and secondary addresses for artists.

- **NextGenAdmins**
```NextGenAdmins``` is responsible for adding or removing global or function-based admins who are allowed to call certain functions in both the Core and Minter contracts. It maintains a mapping of addresses that represent whether a particular address has global admin permissions. The owner of the contract can register or unregister global admins using the ```registerAdmin``` function.

The ```registerCollectionAdmin``` function allows an admin to set permissions for a specific collection. The contract also have functionality to check if an address has global admin permissions ```retrieveGlobalAdmin```, function-specific admin permissions ```retrieveFunctionAdmin``` or collection-specific admin permissions ```retrieveCollectionAdmin```.

- **NextGenCore**
```NextGenCore``` allows for creation of new collections of NFTs, each with its own set of metadata. Each collection is assigned a unique ID and can be updated or frozen. The contract includes functionality for minting new tokens within a collection. Each collection has additional data associated with it, such as the artist's address, maximum purchases, total supply, and other parameters. This data can be set and updated separately from the collection's metadata. The contract also allows for the artist's signature to be added for each collection and for the metadata of a token to be modified.

It also keeps track of various data for each token, such as the collection it belongs to, the hash calculated during minting, and additional metadata. It also keeps track of how many tokens have been minted per address for each collection, both during public sale and during an ```allowlist``` sale.

- **RandomizerNXT**
 ```NextGenRandomizerNXT``` contract store addresses of other contracts like IXRandoms, INextGenAdmins and INextGenCore. The functions ```updateRandomsContract```, ```updateAdminsContract```, and ```updateCoreContract``` allow for updating the addresses of the associated contracts. ```calculateTokenHash``` function calculates a hash value based on the ```_mintIndex```, the hash of the last block, a random number, and a random word. The hash is then set as the token hash for a specific collection and mint index in the ```gencoreContract```.

- **RandomizerRNG**
```NextGenRandomizerRNG``` contract  have ```requestRandomWords``` function to request random numbers from the ```arrngController``` contract. The function requires the caller to be the gencore contract and takes in a tokenid and _ethRequired as parameters. The ```fulfillRandomWords``` function is called when the ```arrngController``` contract has generated the random numbers.It calls the setTokenHash function on the gencoreContract to set the hash for the token.

```calculateTokenHash``` function is used to calculate the hash for a token. It maps the tokenIdToCollection and calls the ```requestRandomWords``` function to request random numbers. ```updateAdminContract``` and ```updateCoreContract``` functions are used to update the addresses of the admin and core contracts respectively. ```updateRNGCost``` function is used to update the cost of requesting random numbers. ```emergencyWithdraw``` function is used to withdraw any balance from the smart contract. 

- **RandomizerVRF**
```NextGenRandomizerVRF``` contract has several state variables, including COORDINATOR, s_subscriptionId, keyHash, callbackGasLimit, requestConfirmations, numWords, gencore, gencoreContract, and adminsContract. The variables store the state of the contract and can be accessed and modified by the contract's functions.

The ```requestRandomWords``` function is used to request random words from the ```COORDINATOR```. The ```calculateTokenHash``` function is used to calculate a hash for a token and request random words. Update functions like ```updatecallbackGasLimitAndkeyHash```, ```updateAdditionalData```, ```updateAdminContract```, and ```updateCoreContract``` allow updating various states of the contract. 

- **XRandoms**
```randomPool``` contract is designed to generate random words from a predefined list as ```getWord``` function takes an integer as an argument and returns a word from the wordsList array. ```randomNumber()``` function generates a random number by using keccak256 hashing function on a combination of the previous block's random number (block.prevrandao) which seems out of context for generating the hash of the block before the current one and current block's timestamp.

```randomWord()``` function is similar to ```randomNumber()``` but it uses the generated random number as an index to pick a word from the wordsList array using the getWord() function. ```returnIndex``` function returns a word from the wordsList array at the given index id using the getWord() function.

# Codebase Quality Analysis
The codebase is simple and uses clearly defined and standard methods to generate randomness for each NXT generated. However some issues in quality of the code are mentioned below

**NextGenAdmins** contract lacks events that would allow tracking of state changes, like an admin is registered or a permission is changed. This could make it harder to monitor and audit the contract's activity. The contract does not have a function to remove an admin. If an admin's private key is compromised, there is no way to revoke their admin status.

**RandomizerNXT**, **RandomizerVRF** and **RandomizerRNG** contracts lack events that would allow tracking of state changes, such as when a contract address is updated. It also does not check if the provided addresses for the randoms, admins, and core contracts are valid (non-zero) addresses.

**XRandoms** contract uses block variables (block.prevrandao, blockhash(block.number - 1), block.timestamp) for generating random numbers and words.It's important to note that the randomness in this contract is not truly random. It's deterministic and can be predicted, which is a security concern in many blockchain applications.
## What can be done differently
Here are some approached that can be taken to improve the quality and security of the code for each contract:

```NextGenAdmins.sol``` need to have more events to the contract. Events allow light clients to react on changes efficiently. Using external instead of public for functions that are not called within the contract. This can save gas and make the code simpler and understandable about its intended use.

```RandomizerNXT.sol``` have ```calculateTokenHash``` function does not have any access control, which means any address can call it. Consider adding a modifier to restrict access.

# Centralization Risks
Centralization risk identified for the contracts in scope are related to lack of transparency, potential manipulation, and a single point of failure.

```NextGenCore.sol``` contract, have several functions that can only be executed by an admin or a specific role, like ```changeMetadataView``` and ```changeTokenData``` function can only be executed by admin. This means that the ability to change metadata view and token data is centralized to specific roles.

```MinterContract.sol``` contract, the ```proposeSecondaryAddressesAndPercentages``` function can only be executed by an artist or admin. This makes the ability to propose secondary addresses and percentages is centralized to the artist or admin.

```NextGenAdmins.sol``` and ``` RandomizerNXT``` contract has the highest degree of centralization as the owner has the ability to register and manage admins, function admins, and collection admins.

```RandomizerRNG.sol``` contract allows for the updating of the admin contract, core contract, and RNG cost only by function admins or global admins. It also has ```emergencyWithdraw()``` function allows the contract owner to withdraw all the balance from the contract. This could be a potential security issue if the owner's account is compromised.

 ```RandomizerVRF.sol``` contract allows for updating the admin contract, core contract, callbackGasLimit, keyHash, and other data only by function admins or global admins. 

# Systemic Risks
Systemic risk identified for the NextGen are identified mainly in areas of security vulnerabilities or design flaws that could potentially disrupt the whole protocol. The risks are identified as below:

There are several functions in all contracts that does not access restricted to certain roles. If the access control is not properly implemented, it could lead to unauthorized access.

In ```AuctionDemo```` users make bids for tokens, it could lead to front-running, where someone sees a transaction in the mempool and quickly submits another transaction with a higher gas price to jump the queue for bid.

The ```NextGenCore``` contract use some randomness in the ```_mintProcessing``` function. If the source of randomness is predictable, it could be manipulated by an attacker.

In ```RandomizerNXT```, functions ```updateRandomsContract```, ```updateAdminsContract```, ```updateCoreContract``` can be misused to set the contract addresses to malicious contracts, which could then manipulate the contract's behavior.

```RandomizerRNG``` have functions ```updateAdminContract```, ```updateCoreContract``` and ```updateRNGCost``` functions that can be misused to set an excessively high RNG cost, which could then manipulate the contract's behavior to cause users to overpay.

```RandomizerVRF``` have functions that have risk related to randomness source manipulated, leading to predictable hashes and potential loss of funds.


# Architecture Recommendations
Recommendation to improve overall codebase of Nextgen is made in area of applying effective security measures and reducing any design flaw that existed in any contract. The recommendation are made as below
Some functions in the contracts seem to be quite complex and could potentially consume a lot of gas. It's recommended to optimize these functions to reduce gas costs.

There exist redundancy in the codebase like ```collectionInfoStructure``` and ```collectionAdditonalDataStructure``` structs in ```NextGenCore.sol``` could potentially be combined into a single struct to reduce redundancy and improve code readability.

Recommendation is also made to add more extensive event logging for all important state change and functions. Events provide a way to react to state changes in a contract and are especially useful for debugging and for reacting to contract state changes.

The main recommendation is made to add detailed ```Natspec``` to all functions especially complex functions. it is also advised to Implement checks to ensure that the provided addresses for the admin and core contracts are valid (non-zero) addresses.



# Conclusion
The time i spent on reviewing and writing this analysis is around 14 hours which includes assessing all the contracts in scope. The time spent is first carried out to idnetfiy the working mechanism of the project which leads to get clear idea of code quality of each contract and areas of improvement that is needed.

The understanding of codebase also assist me in identifying risk pertaining to systemic and centralisation which are reported and helps in making recommendation to improve overall architecture which is necessary for safe and smooth working of the ```NextGen``` protocol.





### Time spent:
14 hours
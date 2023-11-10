# [01] Summary of Codebase

## 1.1 Description

NextGen is a generative NFT project on Ethereum designed to explore more experimental directions in generative art and other non-art use cases of 100% on-chain NFTs. At a high level, NextGen can be viewed as a classic on-chain generative contract with extended functionality. It follows the phase-based, allowlist-based, delegation-based minting philosophy of The Memes and allows the passing of arbitrary data to the contract for specific addresses to customize the outputs. Additionally, it incorporates a variety of minting models, each of which can be assigned to a phase.

## 1.2 Collection's Setup Flow

Admins will create a collection and give collection owners the necessary roles. After that, collection owners will have to set their collection's information, costs, and phases.

### 1.2.1 Creating the Collection in `NextGenCore.sol`

- Admins create a collection using `createCollection()`, which will set up the following:
  - Collection's ID
  - Collection's name
  - Collection's artist
  - Collection's description
  - Collection's website
  - Collection's license
  - Collection's base URI
  - Collection's library
  - Collection's script

### 1.2.2 Setting Collection's Data in `NextGenCore.sol`

- Collection admins will set their collection's data using `setCollectionData()`. They will set:
  - Collection's artist address
  - Collection's maximum number of purchases during the public sale for each address
  - Collection's total supply

### 1.2.3 Setting Collection's Randomizer in `NextGenCore.sol`

- Collection admins will set their collection's Randomizer using `addRandomizer()`. This will be used for generating unique hashes for each NFT of the collection. There are three options available:
  - A Randomizer contract that uses the Chainlink VRF to calculate the tokenHash.
  - A Randomizer contract that uses ARRNG for generating the tokenHash.
  - A custom-made implementation Randomizer contract that uses the token id, the blockchash of the previous block, and a random pool of words and numbers to generate the tokenHash.

### 1.2.4 Signing a Collection in `NextGenCore.sol`

- Optionally, collection's artists are able to sign a collection to assure users that he is, in fact, the artist. This will be done by using `artistSignature()`. Once a collection is signed, its artist's address cannot be changed.

### 1.2.5 Freezing a Collection in `NextGenCore.sol`

- Optionally, collection's admins can freeze a collection using `freezeCollection()`, which will lock the information, data, and metadata of a collection forever.

### 1.2.6 Setting Collection's Costs and Sales Model for Minting Process in `MinterContract.sol`

- Collection admins will set their collection's Randomizer using `setCollectionCosts()`. This will set:
  - Collection's mint cost
  - Collection's end mint cost
  - Collection's rate
  - Collection's time period
  - Collection's sales option
  - Collection's delegate address

#### 1.2.6.1 Different Sale Models

- There are three different sale models:
  - Fixed Price Sale
    - The minting cost is fixed during the minting phase. The rate and time period need to be set to 0 as they do not affect the price.
  - Exponential/Linear Descending Sale
    - Exponential: At each time period (which can vary), the minting cost decreases exponentially until it reaches its resting price (ending minting cost) and stays there until the end of the minting phase. In this model, the starting and ending minting costs, as well as the time period, are set while the rate is 0.
    - Linear: At each time period (which can vary), the minting cost decreases linearly until it reaches its resting price (ending minting cost) and stays there until the end of the minting phase. In this model, all parameters need to be set. In the Linear Model, the rate refers to the value for which the starting price will be reduced during each timestep, e.g., if you set starting minting cost to 1ETH and the rate to 0.1 ETH, during each time period price will be decreasing by 0.1 ETH until it reaches the ending cost.
  - Periodic Sale
    - Mints are limited to 1 token during each time period (e.g., minute, hour, day). The minting cost can either be stable or have a bonding curve (increase) over time. This allows for long-lived or experimental mints. Two options:
      - If the rate is set to 0, then the minting cost price does not increase per minting.
      - If the rate is set, then the minting cost price increases based on the tokens minted. In the Period model, the rate refers to a percentage that is used to calculate the new price based on the starting price and current minting supply, e.g., If the starting cost is 1ETH and the rate is set to 10%, during each minting, the price will be increased by 0.1 ETH.

### 1.2.6 Setting Collection's Minting Phases in `MinterContract.sol`

- Collection admins will set the minting phases the collection will have:
  - Allowlist start time
  - Allowlist end time
  - Public sale start time
  - Public sale end time
  - Allowlist's merkle root

## 1.3 Collection Minting Options in `MinterContract.sol`

### 1.3.1 Airdrop in `MinterContract.sol`

- Collection admins are able to airdrop tokens to a list of recipients by using `airDropTokens()`

### 1.3.2 Mint in `MinterContract.sol`

- Users are able to mint NFTs by using the `mint()` function.
  - During the allowlist period, where users will need to provide a merkle proof of their eligibility to mint. They can use this functionality along with its delegator also.
  - During the public sale period

### 1.3.3 Burn to Mint in `MinterContract.sol`

- Users can burn an NFT of the collection to use it as a mint pass for getting the NFTs of the new collection
  - Admins will need to set up the possible mint passes by using `initializeBurn()`

### 1.3.4 Burn or Swap External to Mint in `MinterContract.sol`

- Users can burn or swap an NFT of an external collection to use it as a mint pass for getting the NFTs of the new collection
  - Admins will need to set up the possible mint passes by using `initializeExternalBurnOrSwap()`

### 1.3.5 Mint and Auction in `MinterContract.sol`

- Admins can mint a new NFT and put it in auction by using `mintAndAuction()`. This auction will be held in `AuctionDemo.sol` with a limited timestamp.

#### 1.3.5.1 Auction
- Users can place bids in an auction by using `participateToAuction()`
- Users can cancel their bids before an auction ends by calling `cancelBid()` and `cancelAllBids()`
- Users or admins can claim an auction when it's finished by calling `claimAuction()`
- All the users that placed bids during the auction will get refunded

# [02] Architecture Improvements
- Consider not allowing admins to change the prices of a collection while the sale window is open.
- Consider sending the NFT of an auction directly to `AuctionDemo.sol` instead of to a trusted wallet which needs to make the approve.
- Consider adding NatSpec comment through the whole codebase.
- Consider adding more testing through the whole codebase.
- Consider using a linter for cleaning up and making the codebase more readable
- Consider using a unique mapping and struct for storing all the information and the data of a collection.
- Consider using a unique mapping and struct for storing all the phases and costs of a collection.
- Consider combining these 3 mappings: `tokensMintedAllowlistAddress`, `tokensMintedPerAddress` and `tokensAirdropPerAddress` into one mapping.
- Consider remaking the whole auction system since it has a lot of problems
  - Storing all the bids from an auction in an array will give issues with the gas amounts needed for placing new bids and claiming auctions
  - Giving users the freedom for being able to cancel bids at anytime without having any kind of fee will result in a lot of fake bids and spoofing
  - Using push payments for refunding the bids when claiming an auction, specially native token push payment, will arise multiple issues
- Consider using fuzzing for testing correctly all the different phases, prices and timestamps
- Consider breaking big functions such as `mint()` into smaller internal function for better readability and reusability

# [03] Centralization risks
Even if admins and collection admin's are trusted roles that use multisig wallets, there are a lot of concerning issues about the big centralization of the project. 

## 3.1 Limit the inputs in all the functions, specially the ones collection admin's use
- Even if they are trusted roles, a good practice is to limit the inputs they can use in functions as arguments. This will reduce the risk if their private keys get drained. 
- Also, this will reduce the probability of making an invalid input such as putting the end time of a public sale before the start. This checks will not affect the users in any way and will contribute to building a more robust codebase.

## 3.2 Artists have too little power for withdrawing their funds
- Artist cannot withdraw the funds their collections generate, only project admins are allowed to call the `payArtist()` function. I think a better approach would be setting all the address and percentages for the payments of the collection's funds before the start of the sale. This way, an artist could withdraw the funds that belong to him without needing an admin to call the function to do it. 


### Time spent:
30 hours
## Description overview of Livepeer Onchain Treasury Upgrade
The NextGen core and mint contracts provides a flexible, adjustable, and scalable functionality. The primary objective of these contracts are to create new Collections and mint NFT-s ERC721.
To achieve this, OpenZeppelin comes to help with the Access control, Chainlink with  Randomizer Contract VRF( there are 2 other contracts for randomizer, ARRng.io rng and from the deployers of NextGen). There is documentations of the protocol -> https://seize-io.gitbook.io/nextgen/, which provides a comprehensive description of the functionality of all contracts and diagrams for the sales mode.

A critical aspect to look is ensuring there is no backdoor for malicious user to give fewer amount or take access control of the protocol.

##	System Overview – scope
1. NextGenCore.sol - Core is where ERC721 tokens are minted, creating new Collections and correspond with MinterContract.sol.

2. MinterContract.sol – This contract calculates the price of the NFT and you can mint, burn, airdrop or swap it.

3. NextGenAdmins.sol -  Access control of the whole protocol using OpenZeppelin “Ownable” and added unique modifier and functions

4. RandomizerNXT.sol, RandomizerVRF.sol, RandomizerRNG.sol – Responsible for creating a unique hash for each token.
 
5. Xrandoms.sol – It is used by RandomizerNXT.sol for generating random hash for the token.

6. AuctionDemo.sol – An a effective way to bid for a NFT token, the user who bids the most, claims the ERC721 token.

## Privileged Roles – There are 3 roles 
Global admin – the administrator of the protocol(which can be changed)

Function admin – Can be given by Global Admin, and provides you with permissions of function that has modifier -> “retrieveFunctionAdmin”

Collections admin – Given by the Global Admin or Function Admin, gives you a power to create new Collections.

Modifiers:  

![2023-11-10_10_49](https://user-images.githubusercontent.com/88289662/282007143-0a43e454-7c88-4139-a127-06416695d651.png)

![image](https://user-images.githubusercontent.com/88289662/282059612-01c92ffd-29d9-471a-a5ba-c3c97b86644d.png)

![image](https://user-images.githubusercontent.com/88289662/282059675-b8dc7763-4efe-44d0-bd53-63f3061a5ca6.png)

We asked the sponsors about the trust of the users who will be responsible for these roles, and this was their response:

![image](https://user-images.githubusercontent.com/88289662/282059733-c3b81420-ba25-4e12-a09e-efb957ad9c72.png)

## Approach

The main objective during the analysis was to gather an in-depth understanding of the codebase and its architecture and provide recommendations for improving it. Given the small codebase size (of 1265 SLoC), the following approach was followed:

In-depth line-by-line analysis of the core contracts – NextGenCore, MinterContract, AuctionDemo, NextGenAdmin
	Research of the randomizers that were given.
	Trying to optimize the particular scope.

## Architecture Description and Diagram

`NextGenCore`  - handles functionality for creating new collections, adding additional data to existing tokens, and managing various aspects of the collections and tokens. It also includes functionality for setting and retrieving various data associated with the collections and tokens, such as the artist’s signature, token metadata, and collection information.

`MinterContract` – Is used for minting tokens in a collections. It includes various functionalities such as setting minting costs, setting collection phases, airdropping tokens, minting tokens, burning tokens to mint new ones, initializing burn to mint for collections, setting primary and secondary splits, proposing and accepting addresses and percentages, paying the artist, updating core and admin contracts, and withdrawing any balance from the smart contract.

I made diagram, that shows which function users can only use. Represents a high-level view of the architecture of them.
All other function you need to have privileges by the admin.
  
![image](https://user-images.githubusercontent.com/88289662/282059766-0b3c3f90-c081-4315-bb33-a7ac7f243db1.png)

At lower lever, The MINT() function does two main things: figures out how much your NFT or NFTs are worth, and makes sure you're paying enough. If the amount is too low, it cancels the process. Once that's sorted, it then uses NextGenCore.mint() to officially create and finalize your NFT or NFTs.

`AuctionDemo` – It allows users to participate in an auction, place bids, cancel bids, and claim the auctioned item. The Contract also includes functions to return the highest bid and the highest bidder for a given auction

`NextGenAdmins` – Is an administration contract that manages permissions for different types of administrator. It extends from the “Ownable” contract, which provides basic authorization control function from OpenZeppelin.

## Codebase quality 
In a nutshell, the codebase is very unreadable, is not well-structured, and it is pretty much centralized not defi protocol.

Here's a more detailed assessment of various aspects of the code quality:

`Documentation`: Excellent work of the documentations, it has everything you want. It is very detailed and clear structure.

`Code comments`: While the documentation is in-depth, there was very little comments in the code, which brings hardness of the auditors that read the code.

`Automated tests`: The codebase has barely any tests for the particular code that is being audit.

## Systemic and centralization risks
1. Centralization risks – The whole protocol terribly centralized, 90% of the functions are for admins. And the admin can withdraw of the money with emergencyWithdrawn() function.

2. Front-running risks – There is no protection of front-running of mint, bidding in auction, which leads to losing funds or NFT for users.

3. Reentrancy risks- There is no modifier for reentrancy and malicious users can reenter the burnToMint function, because there is no guard and the function first mint and after that burn to NFT.

4. No check for data- There is no check for data when admins setts Collections, phases or the returned data of .call() in AuctionDemo.sol

## Recommendations
1.	Making the code more readable – There are many variables that are useless 
E.g. -> CollectionTokenMintIndex is equal to mintIndex, mintIndex is useless and can be removed:

![image](https://user-images.githubusercontent.com/88289662/282059800-3f8934e2-2470-471a-93b0-94d142603c58.png)

All of the variabes are with similar names and it is confusing  ->

![image](https://user-images.githubusercontent.com/88289662/282059855-ad286bd4-9c8e-42f5-ba67-2c1fdc49b82c.png)

2.	Improve Code Comments
While the code includes some comments, consider enhacing their precision and descriptive quality, especially in the core contracts. Detailed explanations of essential function would be particularly beneficial to provide for a deeper understanding of the function.
3.	Add modifiers – Add Reentracy modifier for mint, burn, burnToMint, mintAndAuction, burnOrSwapExternalToMint, from OpenZeppelin. Shoud be added and modifier for front-running, because this will lead to loss of frund on users.
4.	Change the calculation percentages – When dealing with percentage rates in calculations, it's essential to follow a two-step process for accurate representation. If the initial rate is given as a percentage, such as 20%, it's first converted into a decimal by multiplying it by 100. In this example, 20% becomes 0.20.
5.	Changing the percentages of getPrice() - When dealing with percentage rates in calculations, it's essential to follow a two-step process for accurate representation. First multiply the rate by 100. After completing the necessary calculations using the decimal representation, it's crucial to revert to the original percentage format. To achieve this, the final result should be divided by 100. This ensures a seamless transition between the percentage and decimal representations, maintaining precision throughout the calculation process. For instance, if the rate is 20%, the calculation involves multiplying by 100 initially and dividing by 100 at the end, ensuring consistency and accuracy in the final outcome.


## In Conclusion
Overall, the NextGeneration protocol represent innovative way for minting NFTs for collections. As matter of fact the documentations beyond doubt helped extremely well for reading the whole codebase and understand it. But it is far away from decentralized system that blockchain wants to be.











### Time spent:
0 hours
# Analysis
## Preface 
This analysis report is a general overview of my auditing process how I understand this codebase and how I tackle this code finding vulnerabilities. This report will provide detailed understanding of codebase and the things developers are unaware of.
## Approach For Auditing Codebase 
I have used the following approach for auditing the codebase:
1.	Firstly, I have read the codebase and understand the codebase.
2.	Then I have read the documentation of the codebase.
3.	Then I have read the codebase again and understand the codebase.
4.	Then I have read the documentation again and understand the documentation.
5.  Then I compared the codebase and documentation and find the differences.
6.  Find some interesting things that were totally off from documentation.
7.  Then I have started to find vulnerabilities in the codebase.
8.  I started understadning protocol codebase from smaller code contracts with minor complexities to the large code contract with major complexities.
9. Starting from smaller code contracts to larger code contracts helped me to understand the codebase and the protocol in a better way.
10. I tried to visualize the bigger picture of the codebase that how minter contract minting NFTs collections and how core contracts Managing NFTs collections like setting collection URI, setting collection base URI, setting collection royalty, setting collection fee, setting collect. How Randomizer are working with `NextGenCore` core to provide random hash that make every NFT collection unique based on unique random hash.
11. I have also tried to understand the codebase by reading the codebase from the perspective of a developer who is going to use this codebase to create their own NFTs collections and I find so many things that developer assume wrong which I will discuss in the next section (Architecural Imporvements).
12. Hats of to one of the warden who drew the architecture diagram of the codebase. It helped me a lot to understand the codebase and the protocol.

[Link If render fails](https://drive.google.com/file/d/1vQ9wtrEaf_TOaJ_RUAkWqLdEjFwCjsqr/view?usp=sharing)

[Codebase Flow Diagram](https://drive.google.com/file/d/1vQ9wtrEaf_TOaJ_RUAkWqLdEjFwCjsqr/view?usp=sharing)

13. I submitted decent medium findings related to some logical flaws that needed to be mitigated.
14. Then I submitted QA report which contained Low severity findings, NCs, some refactoring suggestions and some architectural improvements.
15. In conclusion, I took 7-8 days in understanding codebase and 2 days for writing reports and and 1 day for writing this analysis report.

## Architectural Improvements

### 1. Contract Structure 
The contract structure to be honest can be improved a lot. All the libraries, interfaces, core & helper contracts are in the same folder. It was in asymetrical manner and wasn't in a proper structure. I have suggested the following structure to the team:

```
contracts
├── core
│   ├── NextGenCore.sol
│   ├── MinterContract.sol
│   ├── NextGenAdmins.sol
│   ├── AuctionDemo.sol
├── Randomizers
│   ├── RandomizerNXT.sol
│   ├── RandomizerRNG.sol
│   ├── RandomizerVRF.sol
|   |── XRandoms.sol
├── Interfaces
│   ├── IMinterContract.sol
│   ├── INextGenAdmins.sol
│   ├── INextGenCore.sol
│   ├── IRandomizer.sol
│   ├── IXRandoms.sol
├── Libraries
│   ├── OZ contracts

```
coming to the codebase, main structre of code wasn't upto the mark really like when I was reading the core contract, The code itself isn't following the standard structure for writing solidity contract. There was mappings, structs, events, modifiers, in arbitrary form in up and down manner no really sequence. Same with all the main contracts.

### 2. What's Unique About This Protocol

I find one thing very intriguing about this codebase and that was using admin control in a very weird way.
I think team did a pretty job in handling the codebase with strict access but fails to handle access control at some things which i submitted in my Findings. 
So, most of the things are in admins hand and it's pretty interesting that NextGenTeam is trying to control most of the things in their way.

### 3. What's using existing pattern 

I find one thing that i looked at and audited again and again that was randomizer, Yes I'm not very familiar with RNG Randiomizer but audited many codebases that have Chainlink VRF and that was the interesting thing to look up at again.

Codebase was using ownable access control of OZ that was very familiar and find some decent things in the implementation.

### 4. Codebase Documentation

There wasn't proper documentation in the whole codebase. Actual codebase documentation was very poor, you can say there was only hint about the functions and logics in the contracts that what this specific function does.
If someone has to depend on code documentation then it's very hard to understand the codebase. I would have suggested the team to write proper documentation of the codebase not in the gitbook but also in the codebase comments.

### 5. Overall Codebase
To be honest, codebase wasn't really ready for the audit like there are basic things that can be imporved based on the development side. Like codebase comments can be imporved, functions can be refactored, structure can be imporved. There are so many minor bugs that real bugs might be hidden because codebase wasn't in its best form. NXT Team should consider doing proper testing and imporiving codebase before going for audit.

## Codebase Quality 

Logics were kept very straight and simple. I didn't find any complex logics in the codebase. 
Even though by skimming through the codebase it was looking like that the codebase is very rough and will have a lot of things to disclose but team did it well in handling the logics but overall codebase quality wasn't upto the mark as discuss in the above section.

### 1. Codebase Testing 

**Overall code base quality = 50%**

Codebase testing was very poor. Only tests were written in hardhat and we can only write unit tests in hardhat.
There was no fuzzing testing or invariant testing to check if the edge cases are still holding up or not.
So, Team should consider writing tests in foundry and do proper fuzzing and invariant testing.
I would suggest team to do following steps to imporve codebase quality.

- **Proper Fuzzing With Echidna Or Foundry**

- **Proper Invariant Testing With FOundry**

- **Unit Testing Of Each & Every Function in Foundry**

## Centralization Risk 

Codebase was very strict in access control. Most of the functions were either control by admins of NextGen Team. Even if the artist have to set collection info he himself can't do it only admin or global admin can do it so that makes it very centralized. 

Most of the codebase is handled by admins that are decalred in `NextGenAdmins.sol` and those admins are are set by NextGen Team and those admins are moslty handling the  stuff.

So, I consider this a very centralised codebase.

## Dangerous Rug pull Mechanism 

There is a sneaky function called `emergencyWithdraw()` i want to disclose and how I consider this as a rug pull that it is in the way that all the funds can be transferred to the admins contract means the team can tranfer the funds into their contract.

Just have a look at this function : 
```solidity
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```

This function is in the Minter Contract which mint all the tokens and collections in this codebase. Obviously we can't mint for free and certainly when minting takes place money is coming in this contract an here's NextGen team played a thing they made a the above function that in case of emergency they will call the this function and fun fact is that it is only callable by only admins means NextGen Team. By executing this function they are transferring funds to admins contract. 

There was a discusion that admins and teams are trusted so there shouldn't be issue about admins or NXT team by there was a lot of rug pull cases where admins rug pull their loyal users for the sake of money.
In this case i won't the admins will be ever trusted in this case because if it was like that so there would be no rug pull cases in the history of crypto.
So, I consider this a rug pull scenario, and it should be mitigated.

## Resources Consumed 

I have consumed the following resources in this audit:

- NextGen Team Documentation On GitBook because it contained some dev comments also
- Codebase Comments
- Learned about randomizers 
- Searched Solodit for some relevant findings 
- Some diagrams drawn by chads Wardens on twitter 

## Total Time Spent 

Day 1-2 : Understanding Codebase
Day 3-4 : Finding weakspots, tagging things with audit tags
Day 4-6 : Finding Vulnerabilities
Day 6-8 : Preparing Reports For submission
Day 8-10: Submitting Vulnerabilties, QA Report & Analysis Report

Total Time Spent in Hours : 80 Hours

### Time spent:
78 hours
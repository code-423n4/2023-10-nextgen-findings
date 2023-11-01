## Title
Missing checks `adminContract` compatible in constructor and `updateAdminsContract`(in `RandomizerNXT` contract

## Detail

There are missing admin contract compatibility checks in the following contract's constructors and a update function is concerning, **as it could result in unexpected scenarios**.

Mistakenly assigning an address of an admin contract with high-level access control to manage the core contracts might lead to unforeseen situations within the core contracts.

*There are 7 instances of this issue*

* [AuctionDemo.sol/constructor()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L36-L40)
* [MinterContract.sol/constructor()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L129-L133)
* [NextGenCore.sol/constructor()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L108-L112)
* [RandomizerNXT.sol/constructor()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L25-L30)
* [RandomizerNXT.sol/updateAdminsContract()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L45-L47)
* [RandomizerRNG.sol/constructor()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L29-L33)
* [RandomizerVRF.sol/constructor()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L39-L45)

---

Using `NextGenCore` contract case as an example code to explain the vulnerability, specifically at [L109](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L109), the contract directly assigns the `adminsContract` without verifying whether the provided `_adminsContract` address aligns with the admins contract compatible with this protocol. (There's no validation like `INextGenAdmins(_adminsContract).isAdminContract() == true`).

```solidity
File: smart-contracts/NextGenCore.sol

108:   constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
109:     adminsContract = INextGenAdmins(_adminsContract);
110:     newCollectionIndex = newCollectionIndex + 1;
111:     _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
112:   }

```

## Tools Used

VS Code: Manual

## Recommended Mitigation Steps

Consider always adding a checks for the compatibility of the crucial contract, `adminContract`.
Appying the validation like the `require` statement in the `updateAdminContract` function, the exmaple below is the `updateAdminContract` of the `NextGenCore` contract

 ```solidity
File: smart-contracts/NextGenCore.sol

323:           require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");

```
### [G-01] Variables in the struct can be packed into few storage slots

### Description:

Here "uint setFinalSupplyTimeAfterMint" were changed in to "uint32 setFinalSupplyTimeAfterMint"
over here uint32 is enough for the "setFinalSupplyTimeAfterMint".
 
Optimizing the storage of variables in a smart contract struct can significantly reduce gas costs. Each storage slot saved eliminates the need for an extra SSTORE operation, which costs 20,000 gas for the initial setting of a struct variable. Subsequent read and write operations on these variables also benefit from reduced gas costs.


```solidity
struct collectionAdditonalDataStructure {
    address collectionArtistAddress;         // 20 bytes
    address randomizerContract;              // 20 bytes
    IRandomizer randomizer;                  // 20 bytes (assuming IRandomizer is an address)
    uint32 setFinalSupplyTimeAfterMint;      // 4 bytes
    uint256 maxCollectionPurchases;          // 32 bytes
    uint256 collectionCirculationSupply;     // 32 bytes
    uint256 collectionTotalSupply;           // 32 bytes
    uint256 reservedMinTokensIndex;          // 32 bytes
    uint256 reservedMaxTokensIndex;          // 32 bytes
    // Total: 8 slots 
}
```

#### GitHub Reference
The proposed changes can be reviewed in the context of the existing code at the following GitHub link: [NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L44).




### [G-02] Rearrange the struct to save storage slot

### Description:

To reduce gas costs and enhance storage efficiency in the code by optimizing the struct data arrangement.

```solidity

struct collectionPrimaryAddresses {
    address primaryAdd1;      // 160 bits
    address primaryAdd2;      // 160 bits
    address primaryAdd3;      // 160 bits
  + bool status;              // 1 bit (can fit alongside primaryAdd3 in a 256-bit slot)
    uint256 add1Percentage;   // 256 bits
    uint256 add2Percentage;   // 256 bits
    uint256 add3Percentage;   // 256 bits
  - bool status;

}
```

#### GitHub Reference
The proposed changes can be reviewed in the context of the existing code at the following GitHub link: [MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L73).



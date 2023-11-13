## LOW

ERC2981 not used, instead of that 
```
    // artists primary Addresses
    struct collectionPrimaryAddresses {
        address primaryAdd1;
        address primaryAdd2;
        address primaryAdd3;
        uint256 add1Percentage;
        uint256 add2Percentage;
        uint256 add3Percentage;
        bool status;
    }

    // mapping of collectionPrimaryAndSecondaryAddresses struct
    mapping (uint256 => collectionPrimaryAddresses) private collectionArtistPrimaryAddresses;
```

is used in MinterContract.sol
In the case not using it, It should be removed as its not ERC2981 compliant. Since the Handling of royalty is
different.

### No Zero Address checks

no zero address checks in important places could have costly consequences
particularly in a MAINNET like Ethereum

In Constructors
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L129

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L108

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L36

In Artist Signature
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L257

### Centralized risk
The lack of a DAO means that decisions like emergencyWithdraw might be taken and the artists lose
their royalty payments
Admin can change the Admin, minter contract to any arbitrary address

________________________________________________________________

## QA

mintIndex is recalculated instead of using existing collectionTokenMintIndex (5 instance)


**1)** calculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L230
recalculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L235
__________________________________________
**2)** calculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L264
recalculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L267
__________________________________________
**3)** calculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L185
recalculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L188
__________________________________________
**4)** calculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L279
recalculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L281
__________________________________________
**5)** calculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L358
recalculation:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L362
__________________________________________





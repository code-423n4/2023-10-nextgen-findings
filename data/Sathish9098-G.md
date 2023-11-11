# GAS OPTIMIZATION

##

## [G-] Optimizing ``collectionAdditonalDataStructure`` struct for gas efficiency

> Saves ``4000 GAS``, ``2 SLOT``

-  ``reservedMinTokensIndex`` stores a minimum value that will not exceed the range of a uint96, which means it can effectively represent values from 0 to 79,228,162,514,264,337,593,543,950,335. Therefore, a uint96 is more than sufficient to store ``reservedMinTokensIndex``.

- ``setFinalSupplyTimeAfterMint`` stores timestamps, and using a ``uint32`` is typically more than sufficient. Many protocols use uint32 for timestamps. Therefore, a ``uint96`` is more than enough to store ``timestamp values``.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54
 
```diff
FILE: 2023-10-nextgen/smart-contracts/NextGenCore.sol

struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
+        uint96 reservedMinTokensIndex;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
-        uint256 reservedMinTokensIndex;
        uint256 reservedMaxTokensIndex;
-        uint setFinalSupplyTimeAfterMint;
+        uint96 setFinalSupplyTimeAfterMint;
        address randomizerContract;
        IRandomizer randomizer;
    }

```

##

## [G-] 

constants for unchanged values 

# GAS OPTIMIZATION

##

## [G-] Optimizing ``collectionAdditonalDataStructure`` struct for gas efficiency

> Saves ``2000 GAS``, ``1 SLOT``

- ``setFinalSupplyTimeAfterMint`` stores timestamps, and using a ``uint32`` is typically more than sufficient. Many protocols use uint32 for timestamps. Therefore, a ``uint96`` is more than enough to store ``timestamp values``.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54
 
```diff
FILE: 2023-10-nextgen/smart-contracts/NextGenCore.sol

struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
        uint256 reservedMinTokensIndex;
        uint256 reservedMaxTokensIndex;
-        uint setFinalSupplyTimeAfterMint;
+        uint96 setFinalSupplyTimeAfterMint;
        address randomizerContract;
        IRandomizer randomizer;
    }

```

##

## [G-] 



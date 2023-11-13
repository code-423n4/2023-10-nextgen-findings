# setFinalSupplyTimeAfterMint should not be changed in NextGenCore.setCollectionData()
Accroding to the annotation:
>     // only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L145C1-L145C100
But in setCollectionData，when total supply is set, setFinalSupplyTimeAfterMint could also be changed which it shouldn't be.
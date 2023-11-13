Contract `NextGenCore`:

1. The contracts don't follow clean code practice: 
	- no difference between private and public state variables -> recommandation: make all state variables private and create getters for them
	- structs should start with capital letter(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L29, https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44)

2. Important functionalities should emit events: `createCollection`, `setCollectionData`, etc.. for future indexers or integrations

3. Checking for Access Control by a function selector can introduce bugs, because in different contracts can be different functions with the same selector, therefore, an address with admin rights in contract A for "transferFrom(....)" when also have access in contract B at the same function, or maybe to another functions which has the same selector. Recommandation: In contract `NextGenAdmins` the selectors should be grouped per address of contract

4. Naming of the parameters are not accurate as in most of the functions (`changeTokenData`, `burnToMint`, `burn`, etc...) `_tokenId` is reffered as uint256 and in `updateImagesAndAttributes`(https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L281) it represents an array of uint256. When reffering to an array it should be named on plural form: `_tokenIds` 

5. In the function `mint` the parameter `mintIndex` is not checked to be between the collection `reservedMinTokensIndex` and `reservedMaxTokensIndex` as burn require that the tokenId to be between those values

6. In the function `burnToMint` the tokenId is not checked to be between collection `reservedMinTokensIndex` and `reservedMaxTokensIndex` as function `burn` requires

Contract `MinterContract`:

1. Admin can empty the entire balance of the MinterContract through `emergencyWithdraw`. This can be used as rug pull. Recomandation: create a system that only allows artist to retrieve their part of the collection

2. Function `payArtist` doesn't check that the payments made are successfull. Instead of making a function that sends money, a design like pull payments is safer and moves gas to the user. Also, this assures of descentralization, as the `payArtist` can be only called by an admin

3. Function `burnOrSwapExternalToMint` doesn't respect CEI

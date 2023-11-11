### [Low] Consider introducing `emergencyWithdraw()` to `AuctionDemo`
In case of emergency, the auction participant fund in AuctionDemo should be withdrawed to prevent further loss.

### [Low] Function selector can clash
Usage of function selector for permission in access control modifier can clash and lead to unexpected address having unexpected permission. Consider implementing access control using clear-defined roles using `Ownable` which is already imported into the smart contract.

Example below will pass because the selector bytes will be the same.
```
function testSelectorClash() public {
    bytes4 coreSelector = coreContract.updateAdminContract.selector;
    bytes4 minterSelector = minterContract.updateAdminContract.selector;

    // @audit test will pass because both selector will be the same
    assertEq(coreSelector, minterSelector);
}
```

### [QA] Function state mutability can be restricted to pure
```
File: NextGenAdmins.sol:83:5:
83 |     function isAdminContract() external view returns (bool) {

File: MinterContract.sol:506:5:
506 |     function isMinterContract() external view returns (bool) {

File: XRandoms.sol:45:5:
45 |     function returnIndex(uint256 id) public view returns (string memory) {

File: RandomizerNXT.sol:62:5:
62 |     function isRandomizerContract() external view returns (bool) {
```

### [QA] Visibility for constructor is ignored. If you want the contract to be non-deployable, making it "abstract" is sufficient.
```
File :AuctionDemo.sol:36:5:
36 |     constructor (address _minter, address _gencore, address _adminsContract) public {
```
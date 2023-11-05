[L1] Ownable in MinterContract is not needed

since the contract is using ArtistOrAdminRequired, and having use any of `onlyOwner`, Ownable is not needed.

```solidity
contract NextGenMinterContract is Ownable {
```
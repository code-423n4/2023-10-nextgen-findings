## [Gas Optimizations](https://github.com/code-423n4/2023-10-nextgen/blob/main/4naly3er-report.md#gas-optimizations)

|     | Issue | Instances |
| --- | :--- | :---: |
| GAS-1 | Use `ERC721A` instead `ERC721` | 4   |
| GAS-2 | Make 3 event parameters indexed when possible | 2   |
| GAS-3| Use hardcode address instead address(this) | 1   |
|     | Use `selfbalance()` instead of `address(x).balance` | 2   |
| GAS-5 | Using assembly to check for zero can save gas | 3   |
|     | Using `bool` for storage incurs overhead | 9   |
| GAS-7 | internal functions not called by the contract should be removed to save deployment gas | 1   |
| GAS-8 | Public Functions To External | 8   |
|     |     |     |
|     |     |     |

## [GAS‑1] Use `ERC721A` instead `ERC721`

`ERC721A` is an improvement standard for `ERC721` tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with `ERC721`, `ERC721A` is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: <ins>https://nextrope.com/erc721-vs-erc721a-2/</ins>.

```
constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L108

```
import "./IERC721.sol";
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L18

```
IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L340

```
IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112

## [G-2] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```
event PayArtist(address indexed _add, bool status, uint256 indexed funds);
event PayTeam(address indexed _add, bool status, uint256 indexed funds);
event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L123C1-L126C78

```
event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L24

## [G-3] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824

Proof Of Concept

```
uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L42

## [G‑4] Use `selfbalance()` instead of `address(x).balance`

Use assembly when getting a contract's balance of ETH. You can use `selfbalance()` instead of `address(x).balance` when getting your contract's balance of ETH to save gas. Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH. *Saves 15 gas when checking internal balance, 6 for external*.

```
uint balance = address(this).balance;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L462

```
uint balance = address(this).balance;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L80

## [G‑5] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```
if (lastMintDate[_collectionID] == 0) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L285

```
if (collectionPhases[_collectionId].rate == 0) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L549

```
if (id==0) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L28C15-L28C16

## [G-6] Using `bool` for storage incurs overhead

Use `uint256(1)` and `uint256(2)` for `true`/`false` to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from `false` to `true`, after having been `true` in the past. See <ins>source</ins>.

Note: missed from bots

```
function initializeBurn(uint256 _burnCollectionID, uint256 _mintCollectionID, bool _status) public FunctionAdminRequired(this.initializeBurn.selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L308

```
(bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L434C6-L438C74

```
function registerAdmin(address _admin, bool _status) public onlyOwner {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L38

```
bool status;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L46

```
(bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116

```
(bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L128

```
(bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```

&nbsp;https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L139

```
event PayArtist(address indexed _add, bool status, uint256 indexed funds);
event PayTeam(address indexed _add, bool status, uint256 indexed funds);
event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L123C1-L126C78

```
event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L24

## [G‑7] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

```
function fulfillRandomWords(uint256 id, uint256[] memory numbers) internal override {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L48C72-L48C72

## [G-8] Public Functions To External

External call cost is less expensive than of public functions.  
Contracts are allowed to override their parents’ functions and change the visibility from external to public.  
The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

```
function burn(uint256 _collectionID, uint256 _tokenId) public {
```

&nbsp;https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L204C14-L204C19

```
function artistSignature(uint256 _collectionID, string memory _signature) public {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L257C14-L257C29

```
function retrievePrimarySplits(uint256 _collectionID) public view returns(uint256, uint256){
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L470C14-L470C35

```
function retrievePrimaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L476C14-L476C52

```
function retrieveSecondaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L488C14-L488C54

```
function retrieveCollectionPhases(uint256 _collectionID) public view returns(uint, uint, bytes32, uint, uint){
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L494C14-L494C38

```
function retrieveCollectionMintingDetails(uint256 _collectionID) public view returns(uint256, uint256, uint256, uint256, uint8, address){
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L500C14-L500C46

```
function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L53C14-L53C32

&nbsp;
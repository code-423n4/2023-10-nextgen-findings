## [G-01] `x + 1` can be replaced with `++x`

There are multiple instances in the code where a storage variable is incremented with `x = x + 1;`. The most gas-efficient method is `++x`.

## [G-02] `gencoreContract` and `gencore` always store the same value, causing redundant storage reads

In all of the randomizer contracts, we have two storage variables `gencoreContract` and `gencore`

```solidity
INextGenCore public gencoreContract;
address gencore;
```

They always share the same value. In `NextGenRandomizerNXT`, this also causes each RNG call access two storage slots holding the same value. Interface types and address types can be casted to each other. Therefore you only need to access one value, and then typecast it to interface/address type for less storage reads.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol

```solidity
function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
    require(msg.sender == gencore); // @audit first storage read
    bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));
    gencoreContract.setTokenHash(_collectionID, _mintIndex, hash); // @audit second storage read, same value always
}
```

Each storage read takes **2100 gas** for the first read (Gcoldsload), and **100 gas** for each subsequent reads (Gwarmacess). By reading into one slot twice, instead of two slots once, **2000 gas** is saved per RNG call.

Further gas can be saved by adopting bot report finding **[G-07]** (caching storage into memory) on top of this technique.

## [G-03] `isCollectionCreated` mapping is unnecessary, causing redundant storage write for each collection

The mapping `isCollectionCreated` returns a boolean telling whether a collection is created.

However, `isCollectionCreated[_collectionID]` evaluates to true if and only if `newCollectionIndex > _collectionID`. This is because `createCollection()`, sets the mapping to true, but also increments `newCollectionIndex` right after.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L130-L141

```solidity
function createCollection(/*...*/) {
    // ...
    isCollectionCreated[newCollectionIndex] = true;
    newCollectionIndex = newCollectionIndex + 1;
}
```

Therefore all instances of `isCollectionCreated[_collectionID] == true` can be replaced with `newCollectionIndex > _collectionID`, and the mapping can be removed completely.

This saves one storage write for every collection created, equates to **21000 gas**.

## [G-04] `collectionAdditonalDataStructure.reservedMinTokensIndex` is redundant

For any collection, the value `reservedMinTokensIndex` can only be set in `setCollectionData()`, and is always equal to `_collectionID * 10000000000`. Then there is no reason to save this value into storage rather than calculating directly.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L155

```solidity
collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000); // @audit can be calculated directly, no need to write into storage
```

The gas impact is that
- Saves one storage write when the collection data is set, equates to **21000 gas**.
- For every single usage of `reservedMinTokensIndex`, eliminates the need for a storage read. This equates to **2100 gas** per instance.

## [G-05] `collectionAdditonalDataStructure.reservedMaxTokensIndex` is redundant

For any collection, the value `reservedMaxTokensIndex` is equal to `reservedMinTokensIndex + totalSupply - 1`. Then all look-ups of this value can be easily calculated using the token's total supply, instead of having to use extra storage.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L156

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L307-L311

```solidity
collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1; // @audit can be calculated, no need to save
```

We have also further proven in **[G-04]** that min index is not needed to be stored, further highlighting the impact of this finding.

The gas impact is that
- Saves one storage write when the collection data is set, equates to **21000 gas**.
- Saves one storage write when `setFinalSupply()` is called and the max index is altered, saving a further **2900 gas**. Also saves a handful of storage reads in the same function.

## [G-06] No need to explicitly set `collectionAdditionalData[_collectionID].collectionCirculationSupply` to zero

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L152

```solidity
collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
```

There is no need to set this value to zero, as it is already zero by default.

Saves **2900** gas for eliminating the needing of a Gsreset.

## [G-07] `collectionAdditonalDataStructure`: `randomizerContract` and `randomizer` are always the same value

In the struct `collectionAdditonalDataStructure`, the two variables `randomizerContract` and `randomizer` are provably identical, as they can only be set in a single function `addRandomizer()`:

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L170-L174

```solidity
function addRandomizer(uint256 _collectionID, address _randomizerContract) public FunctionAdminRequired(this.addRandomizer.selector) {
    require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");
    collectionAdditionalData[_collectionID].randomizerContract = _randomizerContract;
    collectionAdditionalData[_collectionID].randomizer = IRandomizer(_randomizerContract); // @audit no need to store this a second time
}
```

Interface types and address types can be casted to each other. Therefore storing the data once is enough.

Saves **21000 gas** by reducing one storage read. While the idea is similar to **[G-02]**, the impact is far greater, as the randomizer has to be set for *each collection*, as opposed to the former.

## [G-07] `_saltfun_o` is never used

`_saltfun_o` is used as a function parameter in some functions regarding minting or burning. However, it is never actually used. Consider removing this to save gas, and only adding when actually needed.

## [G-08] `NextGenRandomizerVRF`: `requestRandomWords()` has a redundant check

The `NextGenRandomizerVRF` contract has the following function

```solidity
function requestRandomWords(uint256 tokenid) public {
    require(msg.sender == gencore);
    // ...
}
```

However, the only entrypoint of this function is through `calculateTokenHash()` in the same contract. Said function also has the same `required` check, therefore the highlighted check above is redundant.

ChainLinkVRF randomizer only needs to be called once per ``mint()`` function call:

The ``mint()`` function from the ``NextGenMinterContract`` allows for minting of multiple nfts at a time.
The ChainLInkVRF randomizer makes a ``randomWord()`` request to the chainLink oracle for each one of these, when
1 random value returned should be more than enough to generate x amount of random hashes for each minted nft.

Id suggest something like:
```
uint256 randomWord;
for(uint256 i; i < toMintAmount; i++){
uint256 n = uint256(keccak256(abi.encodePacked(randomWord)) + i;
bytes32 hash = keccak256(abi.encodePacked(n));
setHash(hash);
}
```

Malicious validators can reorg multiple function calls coming form admins from the ``NextGenCore`` contract:

Due to the nature of splitting allowance for multiple function calls throughout the contract, mistakes like these are very likely to happen. Because the target chain for deployment is Ethereum main net and not Optimism, a router contract, to ensure proper initialization of nft collections should be used.
The sponsors, mentioned that to initialize multiple collecitons, they would make all multiple calls themselves.

POC:

Admin1 initializes the Mango Nft collection. The NextGenCore's internal counter is currently at 1, so admint1 optimistically expects the Mango collection id to be 1.

Admin2 initalizes a fruit Bats nft collection. This time expecting the collection to be at id 2.
Both via ``NextGenCore.createCollection()``

Further calls, by the admins or collaborating artists who are allowed to call ``NextGenCore.setCollectionData()`` could have their internal accounting messed, by a malicious validator changing the order of function calls.
Resulting in UI & maxSupply, royalties addresses being wrong.
This becomes a permanent error, if ``NextGenCore.freezeCollection(Id)`` is consecuentially called.

A fix to this issue would be creating a Router contract batching all these function calls into 1, to ensure proper initialization of collections.
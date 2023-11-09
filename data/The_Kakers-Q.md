# QA Report

## Summary

|Id|Title|
|:----:|:------|
| L-1 | `NextGenRandomizerVRF` randomizer uses Goerli `keyHash` |
| L-2 | `fulfillRandomWords` sets the token hash as the first fulfilled random number instead of calculating a hash |
| L-3 | Insufficient pseudo-randomness is masked under over-complicated implementation |
| L-4 | `requestRandomWords()` is marked as `payable` in `NextGenRandomizerRNG` but it can't be called with `value` |
| L-5 | `getWord()` returns the same word for different ids |
| L-6 | Predictable pseudo-random values should be proposed by trusted roles |
| L-7 | `returnHighestBidder()` doesn't update the `highBid` attribute on its calculation |
| L-8 | Auction participants will have their funds locked until NFT is claimed |
| L-9 | Airdropped minted tokens for auctions should be transfered to a escrow contract |
| L-10 | The receiver of the funds from `claimAuction()` should be moved to another function |
| L-11 | Use `supportsInterface()` from EIP-165 instead of custom-made checks |
| L-12 | `supportsInterface()` is incorrectly implemented on `NextGenCore` |
| L-13 | `tokenId` JavaScript variable in generative scripts should be quoted |
| L-14 | Collection scripts with indexes 999 and 1000 can't be updated |
| L-15 | It is not possible to add or remove generative scripts |
| L-16 | Malicious scripts can be injected on the generative scripts via the token  |
| L-17 | `burnToMint()` does not check that minting costs were set
| L-18 | Merkle nodes can have 64 bytes length, making internal nodes be reinterpreted as leaf values |
| L-19 | Merkle proofs from `burnOrSwapExternalToMint()` could be used in `mint()` and vicevsersa |
| L-20 | Lack of check for empty artist signature |
| L-21 | Artist can't propose new addresses and percentages |
| L-22 | `acceptAddressesAndPercentages()` can be frontrunned by artists to change settings |
| L-23 | `acceptAddressesAndPercentages()` can approve addresses and percentages that have not been set |
| NC-1 | Unnecessary address `payable` conversions |

## Low Severity Issues

### L-1 - `NextGenRandomizerVRF` randomizer uses Goerli `keyHash`

## Impact

The randomizer will never resolve the words they need to generate the token hash to set for the token id.

## Proof of Concept

`requestRandomWords()` requests words with a `tokenHash` to the `VRFCoordinatorV2` contract: [RandomizerVRF.sol#L54-L55](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L54-L55)

That hash is initialized with its Goerli value:

- [RandomizerVRF.sol#L26](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L26)
- [Chainlink Docs](https://docs.chain.link/vrf/v2/subscription/supported-networks#goerli-testnet)

The `VRFCoordinatorV2` contract ignores invalid hashes:

```solidity
    // Note we do not check whether the keyHash is valid to save gas.
    // The consequence for users is that they can send requests
    // for invalid keyHashes which will simply not be fulfilled.
```

[VRFCoordinatorV2.sol#L386-L388](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.8/vrf/VRFCoordinatorV2.sol#L386-L388)

This leads to the `NextGenRandomizerVRF` being unable to resolve the words to set the token hash.

## Recommended Mitigation Steps

Define the `tokenHash` with a variable in the constructor.

### L-2 - `fulfillRandomWords` sets the token hash as the first fulfilled random number instead of calculating a hash

#### Impact

The sources of entropy for the token hash randomness are greatly reduced.

The implementation expects to consider multiple words (in case it is configured that way), AND the token id for randomness, but it is **only** using the **first** word.

This affects both `RandomizerRNG`, and `RandomizerVRF` contracts.

#### Proof of Concept

Notice how `fulfillRandomWords()` calculates the token hash as `bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))`:

```solidity
    function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
        gencoreContract.setTokenHash(
            tokenIdToCollection[requestToToken[_requestId]],
            requestToToken[_requestId],
->          bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))
        );
        emit RequestFulfilled(_requestId, _randomWords);
    }
```

- [RandomizerVRF.sol#L66](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L66)
- [RandomizerRNG.sol#L49](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L49)

The problem is that `bytes32()` is only taking the left-most 32 bytes of the abi encoded data, which correspond in this case to the first word in the `_randomWords` array.

Subsequent words (in the case of `RandomizerVRF`), and the `tokenId` (`requestToToken[_requestId]`) will not be considered as a source of entropy for the hash.

In fact, it will set the token hash with the same value as the first fullfilled word.

#### Recommendation

Calculate the `keccak256` hash of the packed data. This way the whole pack will be considered for the hash:

```diff
    function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
        gencoreContract.setTokenHash(
            tokenIdToCollection[requestToToken[_requestId]],
            requestToToken[_requestId],
-           bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))
+           bytes32(keccak256(abi.encodePacked(_randomWords,requestToToken[_requestId])))
        );
        emit RequestFulfilled(_requestId, _randomWords);
    }
```

### L-3 - Insufficient pseudo-randomness is masked under over-complicated implementation

`RandomizerNXT` attempts to provide pseudo-randomness via `randoms.randomNumber()` and `randoms.randomWord()` though the `XRandoms` contract to create a token hash:

- [RandomizerNXT.sol#L57](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L57)
- [XRandoms.sol#L35-L43](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L35-L43)

But these functions just provide a false sensation of "more" randomness:

- Both functions rely on `abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp)` which will always return the same result for the same block.
- `randomWord()` uses `getWord()` to return a word from a list, but adds no additional value to randomness, as it will always returned a correlated value to its corresponding `id`, and will only be used to create a hash.
- A subset is returned by the operation `% 1000` or `% 100`, which will unnecessary restrict the values of the random numbers, as they will be used to create a hash.

#### Recommendation

Remove the `XRandoms` contract as it adds unnecessary complexity and provides no additional value to the randomness of the result of `calculateTokenHash()` in `RandomizerNXT`.

In case the random words and numbers are valuable, expose them in some way, as it is currently not possible to retrieve them after the token hash is generated. The same applies to [returnIndex()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L45-L47), which is useless, as there is no stored or emitted data about the `id` a token used to generate its hash.

#### L-4 - `requestRandomWords()` is marked as `payable` in `NextGenRandomizerRNG` but it can't be called with `value`

The function is marked as `payable` and sends `value` on it:

- [RandomizerRNG.sol#L40](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L40)
- [RandomizerRNG.sol#L42](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L42)

It can only be called via `calculateTokenHash()`, which doesn't provide it with `value`:

[RandomizerRNG.sol#L56](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L56)

#### Recommendation

Verify if the function should actually be `payable` and remove the modifier in case it doesn't need it.

In case the core contract is supposed to send it some value, modify the calling functions to include the corresponding `value`.

### L-5 - `getWord()` returns the same word for different ids

`XRandoms::getWord()` is used to return "random" words upon a provided `id`. But it will return the same word for ids `0` and `1`, making the first word have 2x the probability of being returned compared to the others:

```solidity
  if (id==0) {
      return wordsList[id];
  } else {
      return wordsList[id - 1];
  }
```

[XRandoms.sol#L28-L32](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L28-L32)

The `if` clause is not needed, as the function is expected to be called upon the result of a `% 100` operation: [XRandoms.sol#L41-L42](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L41-L42), which will return values [0-99] that can safely be used inside the bounds of the words array.

#### Recommendation

Replace the `if` statement with:

```diff
-  if (id==0) {
-      return wordsList[id];
-  } else {
-      return wordsList[id - 1];
-  }
+  return wordsList[id]; 
```

### L-6 - Predictable pseudo-random values should be proposed by trusted roles

`RandomizerNXT` uses predictable pseudo-random values to set the token hash. 

[RandomizerNXT.sol#L57](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L57)

This can be used by users to predict the resulting token, and mint the most valuable ones, or with the rarest attributes, taking advantage over other legit users.

#### Recommendation

Separate the token hash assigment to a separate function, like the other randomizers do, and only allow trusted roles to execute it to prevent user from abusing it.

### L-7 - `returnHighestBidder()` doesn't update the `highBid` attribute on its calculation

#### Impact

`returnHighestBidder` may return a different highest bidder to win the auction than the actual highest one.

#### Proof of Concept

Note how `highBid` is always `0`, and only the `index` is updated when calculating the highest bidder:

```solidity
    function returnHighestBidder(uint256 _tokenid) public view returns (address) {
->      uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }
```

[AuctionDemo.sol#L92](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L92)

Compare it to how `returnHighestBid()` does it:

```solidity
    function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
->          uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
->                  highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```

[AuctionDemo.sol#L71](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L71)

#### Recommendation

Update the `highBid` value:

```diff
    function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
+               highBid = auctionInfoData[_tokenid][i].bid;
                index = i;
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }
```

### L-8 - Auction participants will have their funds locked until NFT is claimed

Bidders can't cancel their bid after the auction ends: [AuctionDemo.sol#L125](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L125).

They don't have a method to withdraw their bid if they didn't win either.

And they have to wait until the winner or an admin claims the NFT to get their bid back [AuctionDemo.sol#L116](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L116).

This is an unnecessary lock of users funds.

#### Recommendation

Allow non-winning bidders to withdraw their bids after an auction ends.

### L-9 - Airdropped minted tokens for auctions should be transfered to a escrow contract

Tokens put into auction are first airdropped to a specific `_recipient` address.

```solidity
    function mintAndAuction(address _recipient, ...)
        /// ...
        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
```

[MinterContract.sol#L282](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L282)

And then they are transfered from that address to the auction winner:

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        /// ...
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
  ->            IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

[AuctionDemo.sol#L113](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112)

If for any reason the `ownerOfToken` didn't approve the token, moved it, its address is compromised, or decided not to transfer it, the whole transaction will revert, preventing any participant to receive their funds.

#### Recommendation

Mint the airdropped token to a escrow contract with an approval to be transfered upon a call from the `claimAuction()` function.

### L-10 - The receiver of the funds from `claimAuction()` should be moved to another function

Despite being a trusted role, if the owner account is compromised, or is moved to a contract that can't accept Ether, the whole `claimAuction()` will be blocked, preventing bidders from getting their bid back, and the auction winner from getting the NFT.

```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        /// ...
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
->              (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

[AuctionDemo.sol#L113](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113)

#### Recommendation

Move the logic to transfer funds from the auction to a separate function.

### L-11 - Use `supportsInterface()` from EIP-165 instead of custom-made checks

[EIP-165](https://eips.ethereum.org/EIPS/eip-165) defines "a standard method to publish and detect what interfaces a smart contract implements".

But the contracts in scope define custom-made checks that are not standard and make integrations more difficult.

Instances:

- `isAdminContract()`
  - Defined in [NextGenAdmins.sol#L83-L85](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L83-L85)
  - Used in:
    - [MinterContract.sol#L455](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L455)
    - [NextGenCore.sol#L323](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L323)
    - [RandomizerVRF.sol#L95](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L95)
    - [RandomizerRNG.sol#L62](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L62)
- `isRandomizerContract()`
    - Defined in:
      - [RandomizerNXT.sol#L62-L64](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L62-L64)
      - [RandomizerRNG.sol#L89-L91](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L89-L91)
      - [RandomizerVRF.sol#L105-L107](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L105-L107)
    - Used in [NextGenCore.sol#L171](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L171)
- `isMinterContract()`
  - Defined in [MinterContract.sol#L506-L508](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L506-L508)
  - Used in [NextGenCore.sol#L316](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L316)

#### Recommendation

Implement the EIP-165 standard with `supportsInterface()`.

### L-12 - `supportsInterface()` is incorrectly implemented on `NextGenCore`

The current implementation removes the support of `ERC721Enumerable` for the `NextGenCore` contract, as `super.supportsInterface(interfaceId)` will return the supported interface of the right most overriden contract, which is `ERC2981`.

This will break any possible integration that expects the contract to support the interface of `ERC721Enumerable`, including the NFT interface for `ERC721`. This could be for example an NFT lending protocol that expects the NFT contract supports it (in this case `NextGenCore`).

```solidity
    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC721Enumerable, ERC2981) returns (bool) { 
        return super.supportsInterface(interfaceId); 
    }
```

[NextGenCore.sol#L337-L339](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L337-L339)

#### Recommendation

Add the contract support for the `ERC721Enumerable` interface in `supportsInterface()`.

### L-13 - `tokenId` JavaScript variable in generative scripts should be quoted

On `NextGenCore::retrieveGenerativeScript()`, the generative script defines the `tokenId` JavaScript variable as:

```solidity
    string(
        abi.encodePacked(
            "let hash='",
            Strings.toHexString(uint256(tokenToHash[tokenId]), 32),
            "';let tokenId=",
            tokenId.toString(), // @audit not quoted
            ";let tokenData=[",
            tokenData[tokenId],
            "];",
            scripttext
        )
    );
```

[NextGenCore.sol#L456](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L456)

Notice how `tokenId.toString()` is not quoted, nor the [toString()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/Strings.sol#L20) function adds quotes.

This will generate a JavaScript variable with a `tokenId` represented as a number, and not a String.

> The Number.MAX_SAFE_INTEGER static data property represents the maximum safe integer in JavaScript (2^53 – 1).

> Double precision floating point format only has 52 bits to represent the mantissa, so it can only safely represent integers between -(253 – 1) and 253 – 1. "Safe" in this context refers to the ability to represent integers exactly and to compare them correctly. For example, Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2 will evaluate to true, which is mathematically incorrect.

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER

Considering that the contract `tokenId` can grow up to 2^256 - 1, this can lead to an incorrect representation of the `tokenId` and wrong calculation of scripts in the generative scripts in JavaScript.

#### Recommendation

Quote the `tokenId` variable:

```diff
    string(
        abi.encodePacked(
            "let hash='",
            Strings.toHexString(uint256(tokenToHash[tokenId]), 32),
-           "';let tokenId=",
+           "';let tokenId='",
            tokenId.toString(),
-           ";let tokenData=[",
+           "';let tokenData=[",
            tokenData[tokenId],
            "];",
            scripttext
        )
    );
```

### L-14 - Collection scripts with indexes 999 and 1000 can't be updated

`NextGenCore::createCollection()` allows the creation of an unbounded array of generative scripts for the collection: [NextGenCore.sol#L138](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L138).

But it doesn't allow updating scripts with index `999` and `1000`, as the `updateCollectionInfo()` function overuses the `index` attribute to conditionally update other collection properties.

[NextGenCore.sol#L240-L252](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L240-L252)

This makes it impossible to update those values, bricking the contract functionality.

#### Recommendation

The recommendation would be to refactor the `updateCollectionInfo()` function, so that it doesn't use the `index` for multiple purpouses, and allow the update of any element on the scripts array.

Another possible but not recommended solution would be to limit the scripts size to < 999.

### L-15 - It is not possible to add or remove generative scripts

Once the `NextGenCore::createCollection()` sets the generative scripts array, its size can't be changed: [NextGenCore.sol#L138](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L138)

The `updateCollectionInfo()` allows to replace specific scripts, but it doesn't allow to remove them, or add a new one, which makes it more difficult to perform updates, as "deleted" scripts would have to be replaced with an empty value, on multiple transactions. Also newly scripts will have to be "squashed" into existing scripts elements.

#### Recommendation

Allow the `updateCollectionInfo()` function to replace the whole scripts array.

### L-16 - Malicious scripts can be injected on the generative scripts via the token data

The `retrieveGenerativeScript()` adds the `tokenData[tokenId]` value to its generative JavaScript script:

```solidity
    string(
        abi.encodePacked(
            "let hash='",
            Strings.toHexString(uint256(tokenToHash[tokenId]), 32),
            "';let tokenId=",
            tokenId.toString(),
            ";let tokenData=[",
            tokenData[tokenId],
            "];",
            scripttext
        )
    );
```

[NextGenCore.sol#L456](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L456)

The `tokenData[tokenId]` can contain a malicious script that closes the opening square brackets, performs an action, while returning.

An example could be something like: `];// malicious script \n return;[` which will output the following script:

```javascript
  let hash='hash';
  let tokenId=123;
  let tokenData=[];
  // malicious script
  return;
  [];
  // scripttext
```

The malicious script can update previous variables like doing: `hash='otherHash';tokenId=666;`, or `tokenData = ["otherTokenData"];`.

It may also perform an XSS attack injecting code to steal cookies or other kind of attacks depending on the executing environment.

Arbitrary token data can be set upon the minting of a new token during the allow list period: [MinterContract.sol#L201](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L201).

Although the data [has to be contained on a merkle proof](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L220), the malicious script may be obfuscated to not raise any alert when being added to the merkle tree with the minting data.

#### Recommendation

Limit the creation of token data to trusted actors, like the admins.

### L-17 - `burnToMint()` does not check that minting costs were set

Both the [mint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L197) function and the [burnOrSwapExternalToMint()](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L329) function have checks to verify that the `setMintingCosts` for the collection is set.

It is recommended to implement the same check for `burnToMint()` as well.

#### L-18 - Merkle nodes can have 64 bytes length, making internal nodes be reinterpreted as leaf values

Some specific token data passed to the minting functions may be crafted to exploit this edge behavior of the merkle proof contract and bypass verification.

The node is calculated here:

- [MinterContract.sol#L212](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L212)
- [MinterContract.sol#L216](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L216)
- [MinterContract.sol#L348](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L348)

An explanation can be found on the OpenZeppelin contract:

```solidity
 * WARNING: You should avoid using leaf values that are 64 bytes long prior to
 * hashing, or use a hash function other than keccak256 for hashing leaves.
 * This is because the concatenation of a sorted pair of internal nodes in
 * the merkle tree could be reinterpreted as a leaf value.
```

[MerkleProof.sol#L13-L16](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MerkleProof.sol#L13-L16)

#### Recommendation

Add a prefix to the node hash, so that it makes sure the length is always > 64 bytes.

#### L-19 - Merkle proofs from `burnOrSwapExternalToMint()` could be used in `mint()` and vicevsersa

The node in the `mint()` function is calculated as `node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));`

While in `burnOrSwapExternalToMint()` it is `node = keccak256(abi.encodePacked(_tokenId, tokData));`

Since the token data is variable, it can be crafted to make the node from one function match the other.

Since both functions use the same merkle root, the proof would be valid.

- [MinterContract.sol#L212](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L212)
- [MinterContract.sol#L216](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L216)
- [MinterContract.sol#L348](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L348)

#### Recommendation

Include some identifier if the same merkle root will be used for proofs with different data. Also hash the token data.

### L-20 - Lack of check for empty artist signature

There is no check to prevent setting the artist signature as empty. Once the signature is set, it can't be changed and will leave the collection with an empty signature: [NextGenCore.sol#L259](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L259)

#### Recommendation

Considering checking that the artist signature

```diff
    function artistSignature(uint256 _collectionID, string memory _signature) public {
+       require(bytes(_signature).length > 0);
```

#### L-21 - Artist can't propose new addresses and percentages

Artist are required that the `status` of the `collectionArtistPrimaryAddresses` and `collectionArtistSecondaryAddresses` are set to `false`, in order to propose new addresses and percentages:

- [MinterContract.sol#L381](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L381)
- [MinterContract.sol#L395](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L395)

But this also means that they can't propose new ones, once these values have been approved.

The inconsistency can be seen clearly on how both functions try to update the `status` variable to `false`, despite the fact that it will always be `false` in that situation, as the previous `require` prevents it otherwise.

- [MinterContract.sol#L389](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L389)
- [MinterContract.sol#L403](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L403)

This results in artist that can't propose new addresses and percentages once they were set

#### Recommendation

One solution can be to create a new set of storage variables to save temporary proposed values by the artist, that the admins can then approve or not.

#### L-22 - `acceptAddressesAndPercentages()` can be frontrunned by artists to change settings

`acceptAddressesAndPercentages()` has no way to prevent an artist frontrunning the transaction and changing expected addresses and percentages that the admins have reviewed, and are willing to approve.

[MinterContract.sol#L408](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L408)

This could be considered to break one of the [main invariants of the protocol](https://github.com/code-423n4/2023-10-nextgen/tree/main#main-invariants), as the admin would approve addresses and percentages different than the ones they expected (without noticing):

> Properties that should NEVER be broken under any circumstance:
> - Payments can only be made when royalties are set, the artist proposes addresses and percentages, and an admin approves them.

#### Recommendation

One way to solve it is passing a hash that should match the on-chain parameters to be accepted.

#### L-23 - `acceptAddressesAndPercentages()` can approve addresses and percentages that have not been set

There is no check verifying that the addresses and percentages have been proposed by the artist. Nevertheless an admin can "accept" them.

Consider checking that the proposed values are not empty.

## Non-Critical Issues

### NC-1 - Unnecessary address `payable` conversions

Addresses don't need to be converted to `payable` to send them value like `payable(addr).call{value: value}`.

This adds unnecessary complexity to the code. Consider removing the `payable` conversions for the following instances:

- [AuctionDemo.sol#L113](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L113)
- [AuctionDemo.sol#L116](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L116)
- [AuctionDemo.sol#L128](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L128)
- [AuctionDemo.sol#L139](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L139)
- [RandomizerRNG.sol#L82](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L82)
- [MinterContract.sol#L434-L438](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L434-L438)
- [MinterContract.sol#L464](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L464)
# QA Report

## Summary

| id | title |
| --- | --- |
| [L-01](#l-01-burntomint-doesnt-follow-check-effects-interactions) | `burnToMint` doesn't follow check effects interactions |
| [L-02](#l-02-xrandomsrandomword-will-return-little-bit-skewed-result) | `XRandoms::randomWord` will return little bit skewed result |
| [L-03](#l-03-on-chain-randomness-is-not-completely-random-and-can-be-influenced) | on-chain randomness is not completely random and can be influenced  |
| [R-01](#r-01-nextgencoreviewcirsupply-could-be-named-better) | `NextGenCore::viewCirSupply` could be named better |
| [NC-01](#nc-01-no-event-emitted-when-new-bid-joins-auction) | No event emitted when new bid joins auction |
| [NC-02](#nc-02-missing-camel-case-in-function-name) | Missing camel case in function name |
| [NC-03](#nc-03-use-address0-instead-of-literal) | Use `address(0)` instead of literal |
| [NC-04](#nc-04-erroneous-comment) | Erroneous comment |
| [NC-05](#nc-05-same-calculation-done-twice) | Same calculation done twice |
| [NC-06](#nc-06-same-condition-checked-twice) | Same condition checked twice |
| [NC-07](#nc-07-unnecessary--1) | unnecessary `* 1` |
| [NC-08](#nc-08-unnecessary-separation-between-declaration-and-assignment) | Unnecessary separation between declaration and assignment |
| [NC-09](#nc-09-variable-assigned-same-value-in-both-branches) | variable assigned same value in both branches |
| [NC-10](#nc-10-variable-declared-long-before-it-is-used) | variable declared long before it is used |
| [NC-11](#nc-11-unnecessary-check-for-minting-amount) | Unnecessary check for minting amount |
| [NC-12](#nc-12-common-code-can-be-moved-to-functions-in-mintercontract) | Common code can be moved to functions in `MinterContract` |
| [NC-13](#nc-13-unnecessary-stack-allocations) | Unnecessary stack allocations |
| [NC-14](#nc-14-unnecessary-check-in-auctiondemoparticipatetoauction) | Unnecessary check in `AuctionDemo::participateToAuction` |
| [NC-15](#nc-15-event-emitted-when-bid-is-refunded-doesnt-include-bid-amount) | Event emitted when bid is refunded doesn't include bid amount |
| [NC-16](#NC-16-xrandomsrandomnumber-and-randomword-can-be-simplified) | `XRandoms::randomNumber` and `randomWord` can be simplified |

## Low

### L-01 `burnToMint` doesn't follow check effects interactions
With burn to mint you can burn an NFT to mint another.

However in [`NextGenCore::burnToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L218-L220):
```solidity
File: smart-contracts/NextGenCore.sol

218:            _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
219:            // burn token
220:            _burn(_tokenId);
```
The "burn NFT" is burnt after the new one is minted. `_mintProcessing` uses `_safemints` which calls the `onERC721Received` callback. Hence there is a point where execution is handed over to a foreign contract and both the "burn NFT" and the "mint NFT" are owned by the caller.

#### Impact
In the protocol under audit nothing bad can happen because of this. Since the OZ ERC721 lib protects against the reentrancy. However, if other protocols builds on top of this, it could potentially lead to read only reentrancy issues for them.

#### Proof of Concept
PoC using foundry. Save file in test in `hardhat/test/BurnToMintTest.t.sol`, also needs `forge-std` in `hardhat/lib`:
```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.19;

import {Test} from "../lib/forge-std/src/Test.sol";
import {NextGenMinterContract} from "../smart-contracts/MinterContract.sol";
import {NextGenAdmins} from "../smart-contracts/NextGenAdmins.sol";
import {NextGenCore} from "../smart-contracts/NextGenCore.sol";
import {IRandomizer} from '../smart-contracts/IRandomizer.sol';

contract MintBurnReceiver is Test {

  NextGenMinterContract minter;
  NextGenCore core;

  uint256 burnCol;
  uint256 mintCol;
  uint256 burnToken;
  uint256 mintToken;
  bool secondMint;

  constructor(NextGenMinterContract _minter, NextGenCore _core, uint256 _burnCol, uint256 _mintCol, uint256 _burnToken, uint256 _mintToken) {
    minter = _minter;
    core = _core;
    burnCol = _burnCol;
    mintCol = _mintCol;
    burnToken = _burnToken;
    mintToken = _mintToken;
  }

  function burnToMint() public {
    secondMint = true;
    minter.burnToMint(burnCol, burnToken, mintCol, 0);
  }

  function onERC721Received(address , address , uint256 , bytes calldata ) external returns (bytes4) {
    if(secondMint) {
      // here both exists and are owned by the same owner
      assertEq(core.ownerOf(burnToken),address(this));
      assertEq(core.ownerOf(mintToken),address(this));
    }

    return this.onERC721Received.selector;
  }
}

contract BurnToMintTest is Test {

  uint256 burnCol;
  uint256 mintCol;

  NextGenMinterContract minter;
  NextGenCore core;
  NextGenAdmins admin = new NextGenAdmins();

  address delegate = makeAddr('delegate');
  address randomizer = makeAddr('randomizer');

  address artist = makeAddr('artist');

  address ada = makeAddr('ada');

  function setUp() public {
    core = new NextGenCore('Test','Test',address(admin));
    minter = new NextGenMinterContract(address(core), delegate, address(admin));
    core.addMinterContract(address(minter));

    vm.mockCall(randomizer, abi.encodeWithSelector(IRandomizer.calculateTokenHash.selector), new bytes(0));
    vm.mockCall(randomizer, abi.encodeWithSelector(IRandomizer.isRandomizerContract.selector), abi.encode(true));

    burnCol = core.newCollectionIndex();
    core.createCollection('','','','', '', '','', new string[](0));
    core.setCollectionData(burnCol, artist, 100, 100, 100);
    core.addRandomizer(burnCol, randomizer);

    minter.setCollectionCosts(burnCol, 0, 0, 0, 100, 0, address(0));
    minter.setCollectionPhases(burnCol, 0, 0, 1, 101, bytes32(0));

    mintCol = core.newCollectionIndex();
    core.createCollection('','','','', '', '','', new string[](0));
    core.setCollectionData(mintCol, artist, 100, 100, 100);
    core.addRandomizer(mintCol, randomizer);

    minter.setCollectionCosts(mintCol, 0, 0, 0, 100, 0, address(0));
    minter.setCollectionPhases(mintCol, 0, 0, 1, 101, bytes32(0));

    minter.initializeBurn(burnCol,mintCol,true);

    vm.warp(1); // jump one sec to enter public phase
  }

  function testMintAndBurnOwnsBothBurnAndMintTokenAtSameTime() public {
    uint256 burnToken = core.viewTokensIndexMin(burnCol) + core.viewCirSupply(burnCol);
    uint256 mintToken = core.viewTokensIndexMin(mintCol) + core.viewCirSupply(mintCol);

    MintBurnReceiver mbr = new MintBurnReceiver(minter, core, burnCol, mintCol, burnToken, mintToken);
    minter.mint(burnCol,1,1,'',address(mbr),new bytes32[](0),address(0),0);

    mbr.burnToMint();
  }
}
```

#### Recommendation
Consider calling `_burn` before you call `_safemint`. That way the burnt token cannot be used for anything when the NFT mint callback is executed.


### L-02 `XRandoms::randomWord` will return little bit skewed result

[`XRandoms::randomWord`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L40-L43):
```solidity
File: smart-contracts/XRandoms.sol

    function randomWord() public view returns (string memory) {
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
        return getWord(randomNum);
    }
```

The important thing here is that `uint(stuff) % 100` can only ever return a value in the range $[0,99]$.

Now lets take a look at [`XRandoms::getWord`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L15-L33):
```solidity
File: smart-contracts/XRandoms.sol

15:    function getWord(uint256 id) private pure returns (string memory) {
16:        
17:        // array storing the words list
18:        string[100] memory wordsList = [/* list of 100 words */];
26:
27:        // returns a word based on index
28:        if (id==0) {
29:            return wordsList[id];
30:        } else {
31:            return wordsList[id - 1];
32:        }
33:    }
```

The check at the end will skew the distribution returned by the random id in the range $[0,99]$.
```solidity
if (id == 0) {
    return wordsList[id];
} else {
    return wordsList[id - 1];
}
```

If 0 is sent, it returns the 0th word, if 1 is sent it will also return the 0th word, then `id-1` up to 99, where it returns the 98th word. Hence the word at the end, the 99th word will never be picked. No watermelon, more acai.

Thus, if you map out the probability of the words, the 0th word will have the probability $2/100$, 2% since two of the incoming values maps to it. The words on indexes 1-98 will all have $1/100$, 1% chance, and the word at index 99, $0/100$ as no input maps to it.

#### Recommendation
Consider removing the `if` and just returning `wordsList[id]`

### L-03 on-chain randomness is not completely random and can be influenced 
Here's a good read on `block.prevrandao`:

https://soliditydeveloper.com/prevrandao

In essence, validators can influence the result of `prevrandao` to their benefit if it is profitable enough. But since there are two alternatives for proper randomness these can be used when appropriate.

`RandomizerNXT` will give a better user experience as there isn't any delay between minting and the randomness. However should not be used when a lot of value is at stake.

#### Recommendation
Consider only using the `RandomizerNXT` when there isn't a lot of value at stake.

## Recommendations

### R-01 `NextGenCore::viewCirSupply` could be named better

[`NextGenCore::viewCirSupply`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L393-L396):
```solidity
File: smart-contracts/NextGenCore.sol

393:    // function to return the circ supply of a collection
394:    function viewCirSupply(uint256 _collectionID) external view returns (uint256) {
395:        return(collectionAdditionalData[_collectionID].collectionCirculationSupply);
396:    }
```

The shortening of `Circulation` to `Cir` is hard to grasp.

Consider calling it `Circ` if not the whole `Circulation`.



## Informational / Non Critical

### NC-01 No event emitted when new bid joins auction

This forces someone who is participating in the action to constantly check if they are still the leader. An event would be very helpful for off-chain monitoring on auction status.

Consider emitting an event when a new bid joins. Both with the previous highest bid and the new.

### NC-02 Missing camel case in function name

[`NextGenCore::retrievewereDataAdded`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L377):
```solidity
File: smart-contracts/NextGenCore.sol

377:    function retrievewereDataAdded(uint256 _collectionID) external view returns(bool){
```

Missing camel case impacts readability of code.

Consider renaming it to `retrieveWereDataAdded`

### NC-03 Use `address(0)` instead of literal

[MinterContract::mint](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L205):
```solidity
File: smart-contracts/MinterContract.sol

205:            if (_delegator != 0x0000000000000000000000000000000000000000) {
```

`0x0000000000000000000000000000000000000000` is hard to quickly understand, common practice is to use `address(0)` as that is easier to read.

Consider changing `0x0000000000000000000000000000000000000000` to `address(0)`

### NC-04 Erroneous comment

[`NextGenAdmins::retrieveFunctionAdmin`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L69-L71):
```solidity
File: smart-contracts/NextGenAdmins.sol

69:    // function to retrieve collection admin
70:
71:    function retrieveFunctionAdmin(address _address, bytes4 _selector) public view returns(bool) {
```

Comment `function to retrieve collection admin` is wrong as the function returns the function admin.

Consider rewording the comment to `function to retrieve function admin`

### NC-05 Same calculation done twice

The calculation `gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);` is done twice here, first assigning it to `collectionTokenMintIndex` then to `mintIndex`. 

Consider replacing the use of `mintIndex` with `collectionTokenMintIndex` saving a bit of gas and also reducing code duplication.

[`MinterContract::burnToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L263-L267):
```solidity
File: smart-contracts/MinterContract.sol

263:        uint256 collectionTokenMintIndex;
264:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
265:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
266:        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
267:        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
```

Same happens in [`MinterContract::mintAndAuction`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L279-L281):
```solidity
File: smart-contracts/MinterContract.sol

279:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
280:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
281:        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```

### NC-06 Same condition checked twice

[`MinterContract::initializeBurn`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L309):
```solidity
File: smart-contracts/MinterContract.sol

309:        require((gencore.retrievewereDataAdded(_burnCollectionID) == true) && (gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");
```

`gencore.retrievewereDataAdded(_burnCollectionID) == true` is checked twice here, these two will obviously always return the same result.

Consider removing one of the checks.

### NC-07 Unnecessary `* 1`

[`MinterContract::burnOrSwapExternalToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361):
```solidity
File: smart-contracts/MinterContract.sol

361:        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
```

The `* 1` doesn't do anything here.

Consider removing it.


### NC-08 Unnecessary separation between declaration and assignment

Separating assignment and declaration is confusing as it communicates that there are different paths to assigning the values. This is not the case here.

Consider combining the declaration and assignment to one statement.

12 instances in `MinterContract`:

[`MinterContract::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L206-L231):
```solidity
File: smart-contracts/MinterContract.sol
206:                bool isAllowedToMint;
207:                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);

230:        uint256 collectionTokenMintIndex;
231:        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
```

[`MinterContract::burnToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L263-L264):
```solidity
File: smart-contracts/MinterContract.sol

263:        uint256 collectionTokenMintIndex;
264:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
```

[`MinterContract::mintAndAuction`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L278-L279):
```solidity
File: smart-contracts/MinterContract.sol

278:        uint256 collectionTokenMintIndex;
279:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```

[`MinterContract::burnOrSwapExternalToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L358-L359):
```solidity
File: smart-contracts/MinterContract.sol

332:            bool isAllowedToMint;
333:            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);

347:            bytes32 node;
348:            node = keccak256(abi.encodePacked(_tokenId, tokData));

358:        uint256 collectionTokenMintIndex;
359:        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
```

[`MinterContract::payArtist`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L424-L433):
```solidity
File: smart-contracts/MinterContract.sol

424:        uint256 artistRoyalties1;
425:        uint256 artistRoyalties2;
426:        uint256 artistRoyalties3;
427:        uint256 teamRoyalties1;
428:        uint256 teamRoyalties2;
429:        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
430:        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
431:        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
432:        teamRoyalties1 = royalties * _teamperc1 / 100;
433:        teamRoyalties2 = royalties * _teamperc2 / 100;
```

### NC-09 variable assigned same value in both branches

[`MinterContract::burnOrSwapExternalToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L342-L353):
```solidity
File: smart-contracts/MinterContract.sol

342:        address mintingAddress;
...
345:        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
...
349:            mintingAddress = ownerOfToken;
...
35:        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
...
353:            mintingAddress = ownerOfToken;
```

`mintingAddress` is assigned the same value in both branches. This can be confusing as the pattern used indicates that a different value is assigned in each branch.

Consider removing `mintingAddress` and just use `ownerOfToken` instead in code following the snippet above.

### NC-10 variable declared long before it is used

[`MinterContract::getPrice`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L531-L546)
```solidity
File: smart-contracts/MinterContract.sol

531:        uint tDiff;
532:        if (collectionPhases[_collectionId].salesOption == 3) {
        	// ... `tDiff` not used here
540:        } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){
            // ... comment
546:            tDiff = (block.timestamp - collectionPhases[_collectionId].allowlistStartTime) / collectionPhases[_collectionId].timePeriod;
```

`tDiff` is declared long before it is used. Where it is declared hints that the variable is being assigned in the first `if`-block while it is not. It is only used in the second `else if`.

Consider declaring the variable when it is assigned on row `546`.

### NC-11 Unnecessary check for minting amount

[`MinterContract::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L223-L224):
```solidity
File: smart-contracts/MinterContract.sol

223:            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
224:            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
```

The first check of these `_numberOfTokens <= gencore.viewMaxAllowance(col)` is unnecessary as the second one will always cover all scenarios.

Consider removing the first check.

### NC-12 Common code can be moved to functions in `MinterContract`

2 instances in `MinterContract`:

#### delegation
The lines that check for delegation are repeated identically two times:

[`MinterContract::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L206-L211):
```solidity
File: smart-contracts/MinterContract.sol

206:                bool isAllowedToMint;
207:                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
208:                if (isAllowedToMint == false) {
209:                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);    
210:                }
211:                require(isAllowedToMint == true, "No delegation");
```

are the same as:

[`MinterContract::burnOrSwapExternalToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L332-L337):
```solidity
File: smart-contracts/MinterContract.sol

332:            bool isAllowedToMint;
333:            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);
334:            if (isAllowedToMint == false) {
335:            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);    
336:            }
337:            require(isAllowedToMint == true, "No delegation");
```

#### `timeOfLastMint`

[`MinterContract::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L242-L253):
```solidity
File: smart-contracts/MinterContract.sol

242:           uint timeOfLastMint;
243:            if (lastMintDate[col] == 0) {
244:                // for public sale set the allowlist the same time as publicsale
245:                timeOfLastMint = collectionPhases[col].allowlistStartTime - collectionPhases[col].timePeriod;
246:            } else {
247:                timeOfLastMint =  lastMintDate[col];
248:            }
249:            // uint calculates if period has passed in order to allow minting
250:            uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
251:            // users are able to mint after a day passes
252:            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");
253:            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
```

is the same as:

[`MinterContract::mintAndAuction`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L283-L295):
```solidity
File: smart-contracts/MinterContract.sol

283:        uint timeOfLastMint;
284:        // check 1 per period
285:        if (lastMintDate[_collectionID] == 0) {
286:        // for public sale set the allowlist the same time as publicsale
287:            timeOfLastMint = collectionPhases[_collectionID].allowlistStartTime - collectionPhases[_collectionID].timePeriod;
288:        } else {
289:            timeOfLastMint =  lastMintDate[_collectionID];
290:        }
291:        // uint calculates if period has passed in order to allow minting
292:        uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
293:        // users are able to mint after a day passes
294:        require(tDiff>=1, "1 mint/period");
295:        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
```

Consider moving these to a function reducing code duplication.

### NC-13 Unnecessary stack allocations

3 instances in `MinterContract`:

[`MinterContract::mint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L198):
```solidity
File: smart-contracts/MinterContract.sol

198:        uint256 col = _collectionID;
```

[`MinterContract::burnOrSwapExternalToMint`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L341):
```solidity
File: smart-contracts/MinterContract.sol

341:        uint256 col = _mintCollectionID;
```

[`MinterContract::payArtist`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L421-L423):
```solidity
File: smart-contracts/MinterContract.sol

421:        address tm1 = _team1;
422:        address tm2 = _team2;
423:        uint256 colId = _collectionID;
```

This will pull data from calldata and allocate it on the stack. This can cause stack too deep errors and is a bit confusing why it is done. Or if compiler optimizations are used, it can cause stack variables to be assigned to memory.

Consider using the calldata parameters directly as that is more efficient.

### NC-14 Unnecessary check in `AuctionDemo::participateToAuction`

The check `if (auctionInfoData[_tokenid][index].status == true)` is unnecessary in:

[`AuctionDemo::participateToAuction`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L68-L79):
```solidity
File: smart-contracts/AuctionDemo.sol

68:            uint256 highBid = 0;
69:            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
70:                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
71:                    highBid = auctionInfoData[_tokenid][i].bid;
72:                    index = i;
73:                }
74:            }
75:            if (auctionInfoData[_tokenid][index].status == true) {
76:                return highBid;
77:            } else {
78:                return 0;
79:            }
```

Since the loop only writes `highBid` when `auctionInfoData[_tokenid][i].status == true`, `highBid` can only ever be a value where `status == true` or `0`.

Consider removing the unnecessary check. This pattern with looping has other issues as well, as highlighted in the bot report so perhaps a redesign of the pattern used is approriate.

### NC-15 Event emitted when bid is refunded doesn't include bid amount

[`AuctionDemo::claimAuction`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L117):
```solidity
File: smart-contracts/AuctionDemo.sol

117:                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
```

Here it mistakenly has `highestBid` not the actual, bid amount, `auctionInfoData[_tokenid][i].bid`. The highest bid amount is already emitted in the `ClaimAuction` event, thus can be found there if needed.

Consider changing the amount sent in the event to the bid amount, `auctionInfoData[_tokenid][i].bid`.

### NC-16 `XRandoms::randomNumber` and `randomWord` can be simplified

[`XRandoms::randomNumber` and `randomWord`](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L35-L43):
```solidity
File: smart-contracts/XRandoms.sol

35:    function randomNumber() public view returns (uint256){
36:        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
37:        return randomNum;
38:    }
39:
40:    function randomWord() public view returns (string memory) {
41:        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
42:        return getWord(randomNum);
43:    }
```

Here, as you see, `uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp)))` is done twice. Since `randomNumber` does a higher modulo, consider rewriting it like this:

```diff
    function randomNumber() public view returns (uint256){
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
        return randomNum;
    }

    function randomWord() public view returns (string memory) {
-       uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
+       uint256 randomNum = randomNumber() % 100;
        return getWord(randomNum);
    }
```
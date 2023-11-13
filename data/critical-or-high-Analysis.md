A manual testing approach has been taken for the audit.

The code could have a better structure and be less tangled. This is important for transparency, security, and gas-efficacy.

For example, instead of repeating lengthy statements to access a field in a mapping similar to this one:

```
collectionArtistPrimaryAddresses[_collectionID].primaryAdd1 = _primaryAdd1;
collectionArtistPrimaryAddresses[_collectionID].primaryAdd2 = _primaryAdd2;
collectionArtistPrimaryAddresses[_collectionID].primaryAdd3 = _primaryAdd3;
collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
```

Using an internal function like `_setPrimaryAddress(_colId, _addr)` and `_setPercentage(_colId, _percentage)` is better. It will be more concise and clear when reading/auditing the code.

Formulas inside `getPrice()` look completely [overwhelming](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L551) and, unfortunately, have a precision loss bug resulting in incorrect price calculations.

Another example is when similar code blocks are re-used across unrelated functions. It's good to separate those code blocks into internal functions or libraries to simplify the code, thus making it less gas-consuming and error-prone.

So, the check with the delegation management contract is performed inside `mint()` and `burnOrSwapExternalToMint()`, repeating the same code block. 

```
    bool isAllowedToMint;
    isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
    if (isAllowedToMint == false) {
        isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);    
    }
    require(isAllowedToMint == true, "No delegation");
```

External public functions have no re-rentrancy protection, resulting in a high-severity issue when it's possible to mint more NFTs than allowed.

It's bad to rely on on-chain randomness (`block.prevrandao`, `block.number`, `block.timestamp`). Because for some situations, it will produce the same random number. For example, token hashes will be the same when NFTs are minted in the same block.

The auction contract mistakenly loops through all the bids made. This results in a high-severity issue when the winner is unable to claim the auction due to Dos attack.

### Time spent:
24 hours
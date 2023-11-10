The `id` in `getWord` seems off.
        // returns a word based on index
```
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
        }
```
if the input id is 0 it returns 0 but if the input id is 1 it returns (1-1) id 0
https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/XRandoms.sol#L15-L33
I would consider:
```
        require(id > 0 && id <= wordsList.length, "ID must be within the range of the words list.");
        return wordsList[id - 1]; // subtract 1 to align with the zero-indexed array
```
instead of the if statement

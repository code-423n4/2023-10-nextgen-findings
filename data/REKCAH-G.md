https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/MinterContract.sol#L224C1-L225C1

Line 224 condition will cover Line 223 condition. I mean there is no need to check Line223.
```
require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
```
IF line224 condition is TRUE, Line223 will TRUE too, and if Line224 is FALSE, Line223 will be FALSE too.
Save gas by deleting Line223.
https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/MinterContract.sol#L233C17-L233C26
When a user wants to mint some tokens, he should estimate requiered ETH as msg.value.
```
require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
```
But user maybe can't estimate exact value and should put enough msg.value which his transaction not revert. Because the price of a token in some situation is time dependant.So it's better to return rest ETH to him at the end of the function.
Optimization, Fixes and enhancement.

- Optimized and Enhanced Word Retrieval Function.

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/hardhat/smart-contracts/XRandoms.sol#L18

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/XRandoms.sol#L28

- Marking the some functions as `pure` accurately reflects its behavior and can help prevent confusion about whether it reads, making the code more clear and precise.

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/hardhat/smart-contracts/RandomizerVRF.sol#L105
https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/hardhat/smart-contracts/RandomizerRNG.sol#L89
https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/hardhat/smart-contracts/RandomizerNXT.sol#L62
https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/hardhat/smart-contracts/MinterContract.sol#L506C14-L506C30
https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/hardhat/smart-contracts/IMinterContract.sol#L8

-  In Solidity, constructors are, by default, already considered public, and specifying it explicitly is unnecessary.

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/hardhat/smart-contracts/AuctionDemo.sol#L36

PR: https://github.com/code-423n4/2023-10-nextgen/pull/3
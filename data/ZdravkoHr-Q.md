# Report

# [L-1] Malicious receiver can stop airdropping
The MinterContract allows airdrops. The [airdrop](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181-L192) function loops through every recipient and mints him an amount of NFTs. If one of the receivers does not implement the ERC721 **onERC721Received** or implements it to always revert, the whole airdrop will fail. Consider using [Pull over Push](https://fravoll.github.io/solidity-patterns/pull_over_push.html)

# [NC-1] Organize the files in the main folder
Currently, all files are stored in the root folders. Consider organizing them into different folders. For example, **libs**, **randoms**, **core**


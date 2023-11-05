

Emergency withdraw at RandomizerRNG contract (https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol
) does not check if admin is set or not, if not it will retrurn address(0), and it will sends the ethers to address(0) which will be lost forever
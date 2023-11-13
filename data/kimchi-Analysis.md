## General analysis

One of the main problems of this code is the lack of intensive tests that would be able to detect many irregularities and increase its effectiveness. The project relies largely on very high centralization and assumes proper action in good faith (despite this, it creates a granular system of roles by isolating several of them - this is not consistent with such high centralization). Typically, we either have high centralization (and it requires only one role) or we create a granular system of permissions thanks to which we are able to delegate some tasks (such roles can no longer be considered fully trusted).

### Freeze for `collectionWebsite` is not a good design decision.

This creates room for threats such as domain takeover, which could expose both artists and their fans. If someone takes over the specified in collectionWebsite domain after `freezeCollection`, there is no way to update it (and this is not uncommon, as domains expire), so all users will be directed to the malicious domain.

On the other hand, if the artist wants to redirect users to another (potentially not accepted) domain they can still do it using a redirect.

Freezing `collectionWebsite` causes more problems than not freezing it.

**Suggestion**
Freeze only data that should not change in any case after freeze, exclude `collectionWebsite`.

### Don't rely on the infallibility of trusted roles

A lot of things depends on lack of mistakes and good behavior of trusted roles. This opens up new attack vectors and the possibility of damaging the system. Many values are not checked whether they make sense, examples include:

* setCollectionPhases does not check whether `endTime` is later than `startTime`.
* possibility of `renounceOwnership`.
* no 2-step ownership transfer.

**Suggestion**
If you have granular access control, treat less privileged roles as potential attackers and limit the damage they can cause as much as possible.

### Basic tests and comments

Tests are very basic and have very low coverage. It is worth expanding them to include positive scenarios, various mutations, and those covering identified threats and vulnerabilities. More information about what the tests should look like can be found here: https://ethereum.org/en/developers/docs/smart-contracts/testing/ Threat scenarios can be based on the findings from the C4A contest (so that you don't repeat them when making changes in the future) and on standards such as the Smart Contract Security Verification Standard v2.

------------------------------------|----------|----------|----------|----------|----------------|
File                                |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------------------------------|----------|----------|----------|----------|----------------|
 smart-contracts/                   |    16.43 |    11.59 |    24.54 |    16.23 |                |
  Address.sol                       |     3.85 |        0 |     7.69 |      3.7 |... 233,236,241 |
  ArrngConsumer.sol                 |        0 |        0 |        0 |        0 |       23,51,52 |
  AuctionDemo.sol                   |        0 |        0 |        0 |        0 |... 139,140,148 |
  Base64.sol                        |        0 |        0 |        0 |        0 | 25,28,36,39,90 |
  Context.sol                       |       50 |      100 |       50 |       50 |             22 |
  ERC165.sol                        |        0 |      100 |        0 |        0 |             28 |
  ERC2981.sol                       |    33.33 |    16.67 |    28.57 |    23.81 |... 125,128,135 |
  ERC721.sol                        |    26.67 |    14.58 |    35.48 |    26.58 |... 411,414,465 |
  ERC721Enumerable.sol              |    35.71 |    22.22 |    33.33 |    33.33 |... 154,157,158 |
  IArrngConsumer.sol                |      100 |      100 |      100 |      100 |                |
  IArrngController.sol              |      100 |      100 |      100 |      100 |                |
  IDelegationManagementContract.sol |      100 |      100 |      100 |      100 |                |
  IERC165.sol                       |      100 |      100 |      100 |      100 |                |
  IERC2981.sol                      |      100 |      100 |      100 |      100 |                |
  IERC721.sol                       |      100 |      100 |      100 |      100 |                |
  IERC721Enumerable.sol             |      100 |      100 |      100 |      100 |                |
  IERC721Metadata.sol               |      100 |      100 |      100 |      100 |                |
  IERC721Receiver.sol               |      100 |      100 |      100 |      100 |                |
  IMinterContract.sol               |      100 |      100 |      100 |      100 |                |
  INextGenAdmins.sol                |      100 |      100 |      100 |      100 |                |
  INextGenCore.sol                  |      100 |      100 |      100 |      100 |                |
  IRandomizer.sol                   |      100 |      100 |      100 |      100 |                |
  IXRandoms.sol                     |      100 |      100 |      100 |      100 |                |
  Math.sol                          |    16.67 |    11.29 |     7.14 |    11.76 |... 334,335,336 |
  MerkleProof.sol                   |        0 |        0 |        0 |        0 |... 207,212,217 |
  MinterContract.sol                |    30.77 |    19.75 |    27.27 |    31.17 |... 538,554,560 |
  NFTdelegation.sol                 |        0 |        0 |     2.94 |     0.17 |... 9,1201,1203 |
  NextGenAdmins.sol                 |    71.43 |    28.57 |       60 |    61.54 | 39,45,51,52,84 |
  NextGenCore.sol                   |    44.74 |    22.45 |    48.84 |       50 |... 439,462,468 |
  Ownable.sol                       |    33.33 |        0 |    28.57 |    36.36 |... 52,63,71,72 |
  RandomizerNXT.sol                 |       80 |     8.33 |    42.86 |    57.14 |... 42,46,50,51 |
  RandomizerRNG.sol                 |        0 |        0 |        0 |        0 |... 81,82,83,90 |
  RandomizerVRF.sol                 |        0 |        0 |        0 |        0 |... 100,101,106 |
  SignedMath.sol                    |        0 |        0 |        0 |        0 |... 30,31,38,40 |
  Strings.sol                       |    71.43 |       75 |    33.33 |    79.17 | 46,53,54,77,84 |
  VRFConsumerBaseV2.sol             |        0 |        0 |        0 |        0 |105,128,129,131 |
  VRFCoordinatorV2Interface.sol     |      100 |      100 |      100 |      100 |                |
  XRandoms.sol                      |    77.78 |       50 |       75 |    77.78 |          29,46 |
------------------------------------|----------|----------|----------|----------|----------------|
All files                           |    16.43 |    11.59 |    24.54 |    16.23 |                |
------------------------------------|----------|----------|----------|----------|----------------|

**Suggestion**
Increase test coverage to 100% by adding new tests. Check both positive scenarios, mutations, and negative ones (potential attacks). Increase the number of comments and use NatSpec https://docs.soliditylang.org/en/latest/natspec-format.html

### After fixes

After fixes I suggest going additionally through the following SCSVSv2 categories:
* G2: Policies and procedures,
* C6: NFT
* I1: Basic

As they will go beyond the contest scope. They also contain significant threats regarding the web2/off-chain parts and will help prepare for mainnet launch better (https://github.com/ComposableSecurity/SCSVS).

### When it comes to NFTs, you and your users should be prepared for a large number of phishing campaigns

Prepare educational campaigns for your users related to phishing. Explain what it is and how to protect yourself against it.

Resources that might help with it:
https://phishingquiz.withgoogle.com/
https://www.phishing.org/phishing-resources
https://www.hoxhunt.com/blog/how-to-recognize-and-avoid-phishing-attacks
https://help.coinbase.com/en/nft/protect/scams 

### Consolidate data into single storage file

Storing and managing information about collections and tokens across multiple mappings and within two contracts increases the chances of errors. A more efficient organization of these data structures and storage mechanisms can decrease the risk of errors that might lead to security vulnerabilities. 

**Suggestion**
Consolidate all storage and details related to collections and tokens from NextGenCore and NextGenMinterContract into a single storage file with fewer mappings.

### Time spent:
16 hours
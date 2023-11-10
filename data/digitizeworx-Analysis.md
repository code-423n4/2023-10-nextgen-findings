_**Analysis Overview**_

NextGen is a suite of smart contracts for launching generative art NFT projects on Ethereum. It consists of a core ERC721 token contract, a minter contract for handling phased minting logic, admin contracts for access control, and various randomizer contracts. 

_**Architecture**_

NextGen follows a modular architecture by splitting functionality across multiple focused contracts:

- NextGenCore - Core ERC721 token contract for minting and burning NFTs
- NextGenMinter - Handles all minting logic and sales models
- NextGenAdmins - Manages access control for admin roles 
- Randomizer Contracts - Generates random number for each token's art

**Benefits of Modular Architecture**

1. Separation of concerns - Each contract has a single responsibility making the logic easier to understand.

2. Flexibility - Components can be upgraded independently. For example, a new Randomizer contract can be deployed and integrated without changing any other logic.

3. Reduce risk surface - Issues with one contract are contained, and do not impact other components.

4. Avoid "God" contract - A single giant contract that does everything is hard to maintain and secure. 

**Interactions Between Contracts**

The core contracts interact with each other as follows:

- NextGenMinter calls minting functions on NextGenCore
- NextGenCore calls Randomizer to generate token seed
- Both NextGenCore and NextGenMinter check admins via NextGenAdmins

This is coordinated via interfaces like `INextGenCore` without tight coupling between contracts.

**Downsides of Modular Design** 

There's a complexity cost for managing interactions between all the contracts and stitching together functionality across them.

Proper integration testing across contracts is critical to ensure correct end-to-end behavior.

The modular architecture maximizes flexibility, upgradability, and separation of concern - best practices for complex dApps.

_**Access Control**_

Access control is handled by the NextGenAdmin contract which manages admin roles. There are 3 levels of access control:

- Global Admins - Highest privilege, can add other admins
- Collection Admins - Access to functions for specific collections 
- Function Admins - Granular access to specific functions

This allows fine-grained control over permissions. The owner of NextGenAdmins serves as the root admin. 

**Implementation** 

These roles are implemented using address mappings and modifiers:

```solidity

// Global admin addresses
mapping(address => bool) public globalAdmins; 

// Collection admin permissions
mapping(address => mapping(uint256 => bool)) public collectionAdmins;

// Function admin permissions 
mapping(address => mapping(bytes4 => bool)) public functionAdmins;

// Modifier checks global admin
modifier GlobalAdminRequired() {
  require(globalAdmins[msg.sender], "Not global admin");
  _;
}

// Modifier checks collection admin
modifier CollectionAdminRequired(uint256 collectionId) {
  require(collectionAdmins[msg.sender][collectionId], "Not collection admin");
  _;
}

// Modifier checks function admin 
modifier FunctionAdminRequired(bytes4 funcSelector) {
  require(functionAdmins[msg.sender][funcSelector], "Not function admin");
  _;
}

```

**Privileges**

Each role has privileges suitable to its scope:

- Global Admin - Add other global/collection/function admins
- Collection Admin - Admin functions for their collection 
- Function Admin - Specific functions in Core and Minter

**Deploying Admins**

The NextGenAdmin owner serves as the root admin. They can add the initial set of global admins, who can then further delegate roles.

Careful assignment of the root admin is critical to avoid granting unintended access.

Potential risks:
- Compromise of top-level admin could allow granting of permissions to malicious actors. Access to admin keys needs to be tightly controlled.
- Need to ensure admin roles are not accidentally granted during deployment or incorrectly assigned.

_**Randomness**_ 

NextGen has 3 randomizer contracts for generating a random hash or "seed" for each token's art generation. This ensures each token gets unique art.

1. RandomizerVRF - Uses Chainlink VRF for randomness
2. RandomizerRNG - Uses ARRNG service
3. RandomizerNXT - Custom implementation using on-chain data

Relying on Chainlink and ARRNG provides strong randomness but has dependencies. The custom RandomizerNXT provides a fallback with on-chain data.

NextGen uses a modular approach with 3 randomizer contracts:

1. RandomizerVRF

- Uses Chainlink VRF to request verifiable random numbers
- Highly secure source of randomness
- Requires LINK token to pay for requests

2. RandomizerRNG 

- Leverages ARRNG.io oracle service
- Also provides cryptographically strong randomness
- Needs sufficient ETH deposited to pay for requests

3. RandomizerNXT 

- Custom implementation using on-chain data 
- Combines block hash, block timestamp, etc to generate seed
- No dependencies, but less secure than VRF/ARRNG

**Using Multiple Sources**

The rationale for having 3 implementations is:

- Mitigate risk of single point of failure
- VRF/ARRNG provide strong randomness, but have dependencies 
- RandomizerNXT gives a fallback using on-chain data only

The admin can configure which randomizer a collection uses.

**Potential Issues**

- Compromise of VRF/ARRNG could allow predicting seeds
- On-chain data in RandomizerNXT can be manipulated by miners
- Adding sources increases complexity and attack surface

Proper configuration and failover procedures are important to leverage the benefits while mitigating the risks.

Potential risks:
- Compromise of VRF/ARRNG could allow prediction of random seeds. Custom randomizer provides mitigation.
- Custom randomizer relies on recent block hashes which could be manipulatable by miners.

_**Minting Logic**_

The MinterContract handles all the minting logic including phased allowlist minting, merkle proofs, various pricing models etc. This keeps the core ERC721 contract lightweight.

All the complex minting logic is handled by the NextGenMinter contract including:

- Allowlist minting 
  - Merkle proofs
  - Mint limits per address
- Various pricing models
  - Fixed price
  - Declining price over time 
  - Periodic minting
- Burn-to-mint
- Mint-and-auction
- Royalty splits

This keeps NextGenCore simple as the standard ERC721 implementation.

**Benefits**

Separating minting logic provides:

- Modularity - The minting modes can be altered without changing Core
- Flexibility - New modes can be added more easily
- Reduce risk surface - Issues contained in Minter only

**Potential Issues**

- More testing required - Rigorously test all minting modes and phases
- Increased attack surface - Any bug in Minter could break entire minting
- Complexity cost - Added integration work between Minter and Core

Proper test coverage across minting variants is critical. The allowlist logic needs robust protections against bypassing limits.

Potential risks:
- Complex minting logic increases potential attack surface - thorough testing of all minting modes and phases required.
- Need to ensure allowlist minting limits are strongly enforced.

_**On-chain Data**_

NextGen stores metadata and art generation scripts fully on-chain for decentralization. There is an option to freeze collections when done.

- Metadata JSON (name, description, attributes, image URI) 
- The code for generating each token's unique art

This provides full decentralization - the artwork can be reconstructed just from the blockchain.

**Freezing Collections** 

Once a collection is complete, the admin can freeze it permanently. This prevents any future changes.

Freezing happens by flipping a `collectionFreeze` boolean to true for that collection. 

**Benefits**

- Fully decentralized metadata, no dependency on external servers
- Freezing ensures authenticity and scarcity guarantees

**Potential Issues**

- On-chain storage is more expensive than centralized servers 
- Need to ensure freeze is irreversible and can't be bypassed
- Metadata signed by artist could provide additional assurances 

Proper implementation of freezing is critical to delivering on the promises of an immutable collection.

Potential risks:
- Freezing mechanism must ensure collection data is then immutable to prevent metadata spoofing.

**Recommendations**

- Use a timelock mechanism like DAOhaus for delaying/approving critical admin functions to mitigate malicious or accidental use.

- Consider slashing conditions around VRF/RNG oracle manipulation if feasible.

- Formal verification of key functions like allowlist minting would provide highest assurance.

> The main risks stem from centralized admin access and complexity around minting logic. Proper access controls for admin functions is critical.







### Time spent:
29 hours
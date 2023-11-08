1. Admin roles can only be registered on the Admin Contract.


NextGenAdmins.sol it seems to be well-structured, to ensure that only admin roles can be registered on this contract.  AdminRequired modifier restricts access to certain functions. However, you need to make sure that this contract is indeed the Admin Contract. You can do this by modifying the constructor to check if the caller is the owner and that the contract hasn't already been set as an Admin Contract.

Here's a modified version of your contract to ensure that only the owner can set it as an Admin Contract:

// SPDX-License-Identifier: MIT

/**
 *
 *  @title: NextGen Admin Contract
 *  @date: 26-September-2023 
 *  @version: 1.1
 *  @author: 6529 team
 */

pragma solidity ^0.8.19;

import "./Ownable.sol";

contract NextGenAdmins is Ownable {
    // sets global admins
    mapping(address => bool) public adminPermissions;

    // sets collection admins
    mapping(address => mapping(uint256 => bool)) private collectionAdmin;

    // sets permission on specific function
    mapping(address => mapping(bytes4 => bool)) private functionAdmin;

    bool public isAdminContract = false;

    constructor() {
        // Ensure the caller is the owner before setting this contract as an Admin Contract.
        require(_msgSender() == owner(), "Only the owner can set this contract as an Admin Contract");
        isAdminContract = true;
    }

    // certain functions can only be called by an admin
    modifier AdminRequired {
        require(adminPermissions[msg.sender] || (_msgSender() == owner()), "Not allowed");
        _;
    }

    // rest of your contract...
    // ...
    
}

In this modified version, the constructor now checks if the caller is the owner before setting the contract as an Admin Contract. This way, only the owner can designate this contract as the Admin Contract.


2. Global Admins can only be registered by the Admin Contract owner. (NextGenAdmins.sol)


To ensure that the Global Admins can only be registered by the Admin Contract owner, you should modify the registerAdmin function to include a check that only allows the owner to register global admins. Here's the modified function for NextGenAdmins.sol:

// function to register a global admin
function registerAdmin(address _admin, bool _status) public onlyOwner {
    // Ensure that only the owner can register global admins.
    adminPermissions[_admin] = _status;
}

With this modification, only the owner of the Admin Contract can register global admins using the registerAdmin function, ensuring that global admins can only be registered by the owner.



Function and Collection admins can only be registered by global admins. (NextGenAdmins.sol)



To ensure that the Function Admins and Collection Admins can only be registered by Global Admins, you need to add additional access control checks to the registerFunctionAdmin and registerCollectionAdmin functions. Here's the modified NextGenAdmins.sol code with these checks:

// function to register function admin
function registerFunctionAdmin(address _address, bytes4 _selector, bool _status) public AdminRequired {
    // Ensure that only Global Admins can register function admins.
    require(adminPermissions[msg.sender], "Only Global Admins can register function admins");
    functionAdmin[_address][_selector] = _status;
}

// function to register batch functions admin
function registerBatchFunctionAdmin(address _address, bytes4[] memory _selectors, bool _status) public AdminRequired {
    // Ensure that only Global Admins can register function admins.
    require(adminPermissions[msg.sender], "Only Global Admins can register function admins");
    for (uint256 i = 0; i < _selectors.length; i++) {
        functionAdmin[_address][_selectors[i]] = _status;
    }
}

// function to register a collection admin
function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
    // Ensure that only Global Admins can register collection admins.
    require(adminPermissions[msg.sender], "Only Global Admins can register collection admins");
    require(_collectionID > 0, "Collection Id must be larger than 0");
    collectionAdmin[_address][_collectionID] = _status;
}

With these modifications, only Global Admins can register Function Admins and Collection Admins, as they are required to have the adminPermissions[msg.sender] check in these functions. This ensures that these admin roles can only be registered by Global Admins.


3. Specific admin roles can call the functions of the smart contracts.(NextGenAdmins.sol)

To allow specific admin roles to call the functions of the smart contract, you can create additional modifiers for each type of admin (e.g., FunctionAdmin and CollectionAdmin) and use them in the functions that should be accessible only by these admin roles. Here's the modified the NextGenAdmins.sol code: 
// Modifier for Function Admins
modifier FunctionAdmin {
    require(functionAdmin[msg.sender][bytes4(keccak256(msg.data))] || adminPermissions[msg.sender], "Not allowed");
    _;
}

// Modifier for Collection Admins
modifier CollectionAdmin(uint256 _collectionID) {
    require(collectionAdmin[msg.sender][_collectionID] || adminPermissions[msg.sender], "Not allowed");
    _;
}

// function to register function admin
function registerFunctionAdmin(address _address, bytes4 _selector, bool _status) public AdminRequired {
    // Ensure that only Global Admins can register function admins.
    require(adminPermissions[msg.sender], "Only Global Admins can register function admins");
    functionAdmin[_address][_selector] = _status;
}

// function to register a collection admin
function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
    // Ensure that only Global Admins can register collection admins.
    require(adminPermissions[msg.sender], "Only Global Admins can register collection admins");
    require(_collectionID > 0, "Collection Id must be larger than 0");
    collectionAdmin[_address][_collectionID] = _status;
}

// Example function that can only be called by Function Admins
function someFunctionForFunctionAdmins() public FunctionAdmin {
    // Function logic here
}

// Example function that can only be called by Collection Admins for a specific collection
function someFunctionForCollectionAdmin(uint256 _collectionID) public CollectionAdmin(_collectionID) {
    // Function logic here
}

With these modifications, you've created two new modifiers: FunctionAdmin and CollectionAdmin, which check whether the caller has the necessary admin roles to access certain functions. You can apply these modifiers to the relevant functions to control access based on admin roles.


4. Consider ways in which the generator will not produce a hash value, besides the lack of funds on the VRF and RNG.


If the NextGenRandomizerNXT contract is required to generate a hash value regardless of the availability of funds in the VRF and RNG services, then the hash generation process should not be dependent on these services, as they could fail if not funded.

In order to ensure that calculateTokenHash function always produces a hash value, you could use a combination of on-chain and potentially off-chain data to ensure that even if the VRF and RNG services are not available, the contract can still produce a hash.

Below is a version of the calculateTokenHash function that generates a hash even if the randoms contract is not funded:

// SPDX-License-Identifier: MIT

/**
 *
 *  @title: NextGen Randomizer Contract
 *  @date: 18-October-2023 
 *  @version: 1.4
 *  @author: 6529 team
 */

pragma solidity ^0.8.19;

// Import statements and the rest of the contract remain the same...

contract NextGenRandomizerNXT {
    // ...rest of the contract remains the same...

    // A fallback random number in case the VRF and RNG are not available
    uint256 private constant fallbackRandomNumber = 123456789;
    uint256 private constant fallbackRandomWord = 987654321;

    // Function that calculates the random hash and returns it to the gencore contract
    // Now it will always produce a hash, regardless of the funding status of the RNG/VRF
    function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
        require(msg.sender == gencore, "Caller must be the gencore contract");

        // Try to use the randoms contract, but if it fails, use fallback values
        uint256 randomNumber;
        uint256 randomWord;
        try randoms.randomNumber() returns (uint256 num) {
            randomNumber = num;
        } catch {
            randomNumber = fallbackRandomNumber;
        }
        try randoms.randomWord() returns (uint256 word) {
            randomWord = word;
        } catch {
            randomWord = fallbackRandomWord;
        }

        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randomNumber, randomWord, _saltfun_o));
        gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);
    }

    // ...rest of the contract remains the same...
}
In this modified function:

I introduce fallbackRandomNumber and fallbackRandomWord constants as a backup in case the randoms contract fails to provide the required random values.
You could use try and catch statements to attempt to call randomNumber() and randomWord() on the randoms contract. If those calls fail (which might happen if the VRF/RNG services are not funded or if any other error occurs), the catch block sets the random number and word to our predefined fallback values.
The hash calculation now includes an additional _saltfun_o value.



**smart-contracts/NextGenCore.sol**
- L240/248 - In the updateCollectionInfo() function, flow controllers are used validating that _index == 1000 and _index == 999, but it is not explained why those numbers are.
This should be more explanatory, giving it a name according to these values ​​and not just using the numbers without an explanation.


**smart-contracts/MinterContract.sol**
- L196-254 - The mint() function, being so complex and having more than 55 lines of code, should use auxiliary functions to simplify the code and make it easier for a developer to read.

- L212/216/316/327/348 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123 ,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred.” If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

- L249/292/536/546/551/553 - A division is made by collectionPhases[col].timePeriod, without validating if this variable is != 0, this is important since it could generate an unhandled exception.


**smart-contracts/RandomizerNXT.sol**
- L57 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

- L56 - A require is made and no error message is sent, this would be necessary so that the user and developer better understand what is happening.


**smart-contracts/RandomizerVRF.sol**
- L53/72 - A request is made and no error message is sent, this would be necessary so that the user and developer better understand what is happening.


**smart-contracts/RandomizerRNG.sol**
- L41/54 - A request is made and no error message is sent, this would be necessary so that the user and developer better understand what is happening.


**smart-contracts/XRandoms.sol**
- L36/41 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

## [L-01] Shadowing local variables
## Impact
Detection of shadowing using local variables.

Shadowing using local variables in Solidity occurs when a local variable in a function has the same name as a state variable. When this happens, the local variable takes precedence over the state variable within the scope of the function, effectively "shadowing" it. This can lead to bugs if the programmer intended to interact with the state variable but inadvertently interacted with the local variable instead. It's generally recommended to avoid variable shadowing to prevent such confusion and potential errors.

**Contract:** 
```txt
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol
```
**Locations**
```sol
NextGenCore.constructor(string,string,address).symbol (smart-contracts/NextGenCore.sol#108) shadows:
- ERC721.symbol() (smart-contracts/ERC721.sol#87-89) (function)
- IERC721Metadata.symbol() (smart-contracts/IERC721Metadata.sol#22) (function)
`constructor(string memory name, string memory `symbol`, address _adminsContract) ERC721(name, symbol) {`
```
```sol
NextGenCore.constructor(string,string,address).name (smart-contracts/NextGenCore.sol#108) shadows:
- ERC721.name() (smart-contracts/ERC721.sol#80-82) (function)
- IERC721Metadata.name() (smart-contracts/IERC721Metadata.sol#17) (function)
`constructor(string memory `name`, string memory symbol, address _adminsContract) ERC721(name, symbol) {`
```
## Recommendation
Rename the local variables that shadow another component.
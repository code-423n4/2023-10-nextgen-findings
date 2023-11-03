# The `retrieveGenerativeScript` method of `NextGenCore` contract is vulnerable to `XSS` via the `tokenData[tokenId]`.

## Vulnerable Code

```solidity
contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
    ...
    // function to retrieve the Generative Script of a token
    function retrieveGenerativeScript(uint256 tokenId) public view returns(string memory){
        _requireMinted(tokenId);
        string memory scripttext;
        for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
            scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i])); 
        }
        return string(abi.encodePacked("let hash='",Strings.toHexString(uint256(tokenToHash[tokenId]), 32),"';let tokenId=",tokenId.toString(),";let tokenData=[",tokenData[tokenId],"];", scripttext));
    }
    ...
}
```
[Link](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/NextGenCore.sol#L456)

## Explanation

Generation of the `scripttext` variable is not properly sanitized, allowing an attacker to inject malicious code when
minting a token via `_tokenData` param of `NextGenMinterContract.mint` for example.
This can lead to `XSS` or `NodeJS code injection` depending on the context where the `retrieveGenerativeScript` 
function is called.

## POC

```javascript
let tokenToHash = {};
let tokenData = {};

const retrieveGenerativeScript = tokenId =>
    `let hash='${tokenToHash[tokenId]}';let tokenId=${tokenId};let tokenData=[${tokenData[tokenId]}];`;

(async () => {
    const tokenId = 0x2600;
    tokenToHash[tokenId] = '0x7ef493727fe4435f0ae05ee471f4626baefb94f187042d2724983cc2f6cf76b1';
    tokenData[tokenId] = '(()=>console.log("rekt"))()';
    eval(retrieveGenerativeScript(tokenId));
})()
    .catch(e => console.error(e))
    .finally(() => process.exit());
```

## Remediation

Creating a script in a Solidity contract is novel and interesting, but at this time there is no library to sanitize
a user input. The easiest way to avoid this vulnerability is to not allow users to input data that will be used to
create a script.
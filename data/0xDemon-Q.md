# Visibility for constructor is ignored

The issue is regarding the visibility of the constructor in Solidity. In this case, the constructor is declared as **`public`**, which means it can be accessed from any external contract or account. Consider making it "abstract", its sufficient. 

```solidity
FILE : smart-contracts/AuctionDemo.sol

36 constructor (address _minter, address _gencore, address _adminsContract) public {
37        minter = IMinterContract(_minter);
38        gencore = _gencore;
39        adminsContract = INextGenAdmins(_adminsContract);
40    }
```

[ [36 - 40](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L36-L40) ]
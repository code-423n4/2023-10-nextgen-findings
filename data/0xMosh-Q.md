# N-01 : Constructor cant be public ! 
Visibility for constructor should be  ignored  .
```solidity 
 constructor (address _minter, address _gencore, address _adminsContract) public { //@ci  public  ? 
        minter = IMinterContract(_minter);
        gencore = _gencore;
        adminsContract = INextGenAdmins(_adminsContract);
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L36
## recommendation 
Remove `public` visibility  from the constructor . 

# N-01 : Constructor cant be public ! 

```solidity 

```

## recommendation 

# N-01 : Constructor cant be public ! 

```solidity 

```

## recommendation 

# N-01 : Constructor cant be public ! 

```solidity 

```

## recommendation 
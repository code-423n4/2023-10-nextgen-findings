## \[L-01\] The merkle root won’t revert if is set to `0x00`

If merkle root is set to `0x00` it could leave the system in an insecure state where the system will accept any message that it has never seen before and process it as if it were genuine.

This has happened before in the [<ins>nomad bridge incident</ins>.](https://medium.com/numen-cyber-labs/nomadic-bridge-attack-incident-analysis-94716b93fa61)

> The Nomad team initialized the trusted root to be `0x00`. To be clear, using zero values as initialization values is a common practice.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

## \[L‑02\] Use Ownable2Step rather than Ownable

`Ownable2Step` and `Ownable2StepUpgradeable` prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.

```
contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L22

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L20

&nbsp;

# Non-Critical Findings

## \[N-01\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-02\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N-03\] Use underscores for number literals

```diff
+ uint32 public callbackGasLimit = 40_000;
- uint32 public callbackGasLimit = 40000;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L27

## \[N-04\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-05\] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

## \[N-06\] Function writing that does not comply with the Solidity Style Guide

Context  
All Contracts

Description  
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor  
receive function (if exists)  
fallback function (if exists)  
external  
public  
internal  
private  
within a grouping, place the view and pure functions last

## \[N-07\] Generate perfect code headers every time

Description:  
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.
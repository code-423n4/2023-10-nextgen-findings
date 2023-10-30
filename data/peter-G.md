### For-loop not gas efficient

Instances, such as below:

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/NextGenAdmins.sol#L51

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/AuctionDemo.sol#L69

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/AuctionDemo.sol#L90

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/AuctionDemo.sol#L110

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/AuctionDemo.sol#L136

For-loops are given as such:

```
for (uint256 i=0; i<_selector.length; i++) {
    functionAdmin[_address][_selector[i]] = _status;
}
```
## Optimization

Use the unchecked{} syntax, and pre-increment, e.g.:
```
for (uint256 i=0; i<_selector.length;) {
    functionAdmin[_address][_selector[i]] = _status;
    unchecked {
        ++i;
    }
}
```
## Findings Summary

| ID     | Title                                                                       | Severity |
| ------ | --------------------------------------------------------------------------- | -------- |
| [L-01] | Funds being pushed instead of pulled which can lead to unwanted behavior    | Low      |
| [L-02] | getWord() function in XRandoms contract is missing constraints for id input | Low      |
| [L-03] | Code does not follow the best practice of check-effects-interaction         | Low      |

## [L-01] Funds being pushed instead of pulled which can lead to unwanted behavior

In the ['AuctionDemo'](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol) contract claimAuction() function determines the winner of the auction or if no one matches the conditions of the if statement all the bidders are refunded by pushing the funds to all bidders.

[AuctionDemo.sol#L105-L119](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L105-L119)

```solidity
        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
        uint256 highestBid = returnHighestBid(_tokenid);
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
        address highestBidder = returnHighestBidder(_tokenid);
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                to match pull over pull practise
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
```

## Impact

Pushing the funds to multiple or many users can lead to reverting all of the TX's and users being unable to receive their funds back since there is no withdraw() function.

## Proof of Concept

1. Alice place a bid from a contract with 1 ETH's.
2. Bob place a bid with 10 ETH's.
3. If the conditions in the first if statement in claimAuction() are not met all the bids are funded back to the bidders.
4. Alice contract refuses to receive the funds and tx will revert
5. Bob will not receive his funds back leading to being stucked in the contract.

## Recommended Mitigation Steps

Use pull over push practise by implementing withdraw() function for the bidders so when claimAction() has ended the lost bidders be able to withdraw their funds.

## [L-02] getWord() function in XRandoms contract is missing constraints for id input

In ['XRandoms.sol'](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol) `getWord()` function returns a word equal to the `id` input. In the function there is a string array with 100 words. The problem is that there is no check of the `id` input if it exceeds the 100 hundred words and if it does the function calling getWord with invalid id it will revert.

## Impact

`randomWord()` and `returnIndex()` functions , which are in the same contract `XRandoms.sol`, use `getWord()` as a return in the same contract `XRandoms.sol`, but these two functions are called in many places in the whole protocol. By missing a validation check for the `id` input, these functions will revert leading to unwanted behaviour.

## Proof of Concept

```solidity
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }

```

## Recommended Mitigation Steps

Add the following check in `getWord()`:

```diff
function getWord(uint256 id) private pure returns (string memory) {
+  require(id >=0 && id <= 100)
}
```

## [L-03] Code does not follow the best practice of check-effects-interaction

Code should follow the best-practice of CEI, where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs

[NextGenCore.sol#L204-L209](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L204-L209)

[NextGenCore.sol#L213-L223](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L213-L223)

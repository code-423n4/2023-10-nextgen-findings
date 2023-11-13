# Exponential decrease sale model cannot be implemented as described in the docs, and a lower price is consistently charged instead

## Impact
Exponential decrease sale model charges a lower price than it would be expected

## Proof of Concept
Given a collection configured like so, as defined in the docs under 'exponential
decrease' sale model

```solidity
uint256 private constant TIME_PERIOD = 600;
uint256 private constant START_PRICE = 4 ether;
uint256 private constant END_PRICE = 1 ether;
uint256 private constant PERIODS_IN_SALE = 11;
uint256 private START_TIME;

START_TIME = block.timestamp + 100;
minter.setCollectionCosts({
    _collectionID: RASCALS_ID,
    _collectionMintCost: START_PRICE,
    _collectionEndMintCost: END_PRICE,
    _rate: 0, // exponential decrease
    _timePeriod: TIME_PERIOD, // 'tick' every 10 minutes
    _salesOption: 2, // Descending
    _delAddress: rascalsAdmin
});
vm.prank(rascalsAdmin);
minter.setCollectionPhases({
    _collectionID: RASCALS_ID,
    _allowlistStartTime: 0,
    _allowlistEndTime: 0,
    _publicStartTime: START_TIME,
    // end after 11 timeperiods, after when it's defined in the doc's graph
    _publicEndTime: START_TIME + PERIODS_IN_SALE * TIME_PERIOD,
    _merkleRoot: bytes32(0)
});
```

one would expect the mint price to follow the curve defined in the docs,
asserted in foundry like so:

```solidity
function test_THEN_getPrice_follows_price_defined_in_docs() public {
    // ensure we're inside the timeperiod and not in its edge
    vm.warp(START_TIME + 100);
    uint256 startTime = block.timestamp;
    uint256[3] memory curve = [
        uint(4 ether),
        uint(2 ether),
        1 ether + uint(1 ether) / 3
    ];

    for (uint256 i = 0; i <= 10; i++) {
        vm.warp(startTime + i * TIME_PERIOD);
        uint256 price = i > 2 ? END_PRICE : curve[i];
        assertEq(minter.getPrice(RASCALS_ID), price);
    }
}
```

But a lower price would be charged on mint, due to the `decreaserate` being
subtracted from `price` on line 560. That logic was presumably added in order to
follow another sale model, and is interfering with the exponential decrease one.

## Tools Used
Manual review + writing tests in Foundry

## Recommended Mitigation Steps

- Thoroughly tests the smart contracts with different sale model configurations
- Implement the minting phases as a state machine so logic for one stage cannot
  interfere with another

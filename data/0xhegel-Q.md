// Just add a low risk finding for `auctionDemo` contract

## A user can alter the auction winner right before the end time by offering a slightly higher price

## Suggestions 
- Add a delay time when there is new bid just before the end time.
  EG: Introduce a 15-minute delay if a new bid is placed in the last 5 minutes

## Other optimizations for auction
- The price can only be raised by a fixed percentage
EG: The next bid must be at least 10% higher than the previous one.


- Include a withdraw function to retrieve potentially locked ETH, for instance, if someone inadvertently sends ETH directly.
 With the time-lock feature, this can be considered trustworthy

There are a lot of messages about this codebase which, imo, are wrong.
The AuctionDemo contract is wrong in may places and seems to be written by someone else but besides that the codebase is fine.
Some architecture and written choices are not common but not wrong.

It's a classic NFT mint and auction system designed for artist to bring out tokens without being bothered with code or security.
The artist is able to choose different sales methods. The weird part is that the admin is able to perform airdrops and/ or auctions which should be in control of the artist.
Next to the classic design they have implemented a ticket system where one would need to burn a nft in order to mint a nft.
They've implemented randomizers for the tokenhash which seem vulnerable at first sight but are fine since it's just for the hash and other variables are included to create it.

### Time spent:
24 hours
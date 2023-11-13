### NextGenRandomizerNXT.calculateTokenHash never uses the salt
_saltfun_o is passed via NextGenCore._mintProcessing but nothing is done with it.

### Unnecessary variable set
MinternContract.proposePrimaryAddressesAndPercentages and proposeSecondaryAddressesAndPercentages set `collectionArtistPrimaryAddresses[_collectionID].status` to false. This is not required since the status needs to be false in the require statement `require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");`
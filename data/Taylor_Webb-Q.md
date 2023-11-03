There is no need for the mapping `tokenIdsToCollectionIds` within the `NextGenCore.sol` contract:
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L68

Since token IDs are organized by collection ID, the collection ID a token ID belongs to can be deterministically calculated.

Some examples to illustrate:
Token ID 10000000000 => Collection 1
Token ID 10000000001 => Collection 1
Token ID 10000000002 => Collection 1
Token ID 20000000000 => Collection 2
Token ID 20000000001 => Collection 2
Token ID 20000000002 => Collection 2

So, taking advantage of solidity's integer division rounding, we can state:
collection ID = token ID / 10000000000;

And therefore can just do this calculation to determine which collection  a token belongs to, without needing to use a storage mapping.

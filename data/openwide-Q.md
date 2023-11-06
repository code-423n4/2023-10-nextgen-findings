## Probably funds send to wrong address or confusing data inside event

In `RandomizerRNG.sol`, [`emergencyWithdraw()` function](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L79) sends money to owner of `adminsContract` as seen [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L82). 

However, after that an event is emitted `emit Withdraw(msg.sender, success, balance);` which includes person who initiated the withdrawal. Probably you meant to include address of wallet that received the funds, not `msg.sender`. 

The same issue is also valid for `MinterContract.sol`.

I presume that because this event [definition](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L126) is `event Withdraw(address indexed _add, bool status, uint256 indexed funds);`, there is also `event PayTeam` which has `address indexed _add` as first argument, but this argument is responsible for [specifying the recipient](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L442) of the funds.

If your intentions were to send emergency withdrawal funds to the person who called the functions, this issue could be treated as Medium due to potential loss-of-funds. 
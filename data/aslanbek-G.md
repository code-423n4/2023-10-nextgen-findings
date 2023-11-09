[G-01] Rewrite MinterContract#airDropTokens
Indexes should be checked outside the main loop, all at once, via summing ... 

OR add parameter uint256 sumOfTokens
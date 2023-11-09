[L-01] Mints in the first second of the public-only sale will revert

MinterContract#mint uses non-strict inequality, and getPrice uses strict one. 

Because there's no mention of the restrictoins for allowlistStartTime and allowlistEndTime, and because the test file uses allowlistStartTime = allowlistEndTime = publicStartTime, it is reasonable to assume that it would have occured in production. 
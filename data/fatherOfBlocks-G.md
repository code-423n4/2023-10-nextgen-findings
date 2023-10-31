**smart-contracts/NextGenCore.sol**
- L117/124/148/158/171/239/259/267/274/283/293/316/323/345/348 - Instead of validating if(bool == true), you should directly validate if(bool ) and if(!bool).


**smart-contracts/MinterContract.sol**
- L137/144/151158/171/182/197/211/259/277/309/317/328/329/337/416/455 - Instead of validating if(bool == true), you should directly validate if (bool) and if(!bool).


**smart-contracts/NextGenAdmins.sol**
- L32 - Instead of validating if(bool == true), if(bool) and if(!bool) should be directly validated.


**smart-contracts/RandomizerNXT.sol**
- L35 - Instead of validating if(bool == true), if(bool) and if(!bool) should be directly validated.

- L55 - An input is requested that is never used, _saltfun_o.


**smart-contracts/RandomizerVRF.sol**
- L48/95 - Instead of validating if(bool == true), if(bool) and if(!bool) should be directly validated.

- L71 - An input is requested that is never used, _saltfun_o.


**smart-contracts/RandomizerRNG.sol**
- L36/62 - Instead of validating if(bool == true), if(bool) and if(!bool) should be directly validated.

- L53 - An input is requested that is never used, _saltfun_o.

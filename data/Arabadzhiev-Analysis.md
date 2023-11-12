# Approach taken in evaluating the codebase

After getting to know the documentation I went straight into the code base. I first did one quick high level walk around its external functions, just to get to know the main actions that can be performed within it. Just by doing that, I managed to spot 1 high and 1 medium severity issue. After that, it was just a matter of digging deep into the code and thinking of different exploit scenarios, while using the protocol documentation and unit tests (although there weren't many) in order to clear up any ambiguities that I had while performing the audit. 

# Suggestions to the protocol team

## Unit Tests
At its current state, the protocol has way too low test coverage. In my opinion, you should strive to get that up to at least 90% before deploying the protocol to Mainnet, in order to catch any deep logical errors that might have been missed during this audit contest. 

Additionally you should also create a better testing infrastructure, both for yourself and for security researchers that might audit your code in the future, since as of now, writing tests, especially for some of the contracts that don't have any tests at the moment, is a headache.

And since your test coverage is still relatively low and you don't have that many tests, you might want consider the option of switching to `Foundry`, which arguably, is better than `Hardhat`.

## Code formatting
At its current state, the code of the protocol is not formatted properly. You should consider using some sort of a code formtter such as `Prettier`, in order to fix that. Additionally you might also want to look at adding some kind of a lint job to your CI/CD pipelines that are being ran when a new pull/merge request is created, in order to enforce a particullar code style to your code base.

## Code duplication
At its current state, the code of the protocol has a considerable ammount of duplicaiton within it. You should definetelly try to iliminate that, especially if you intend to make some significant changes to the code prior to deploying the smart contracts on Mainnet. That way, you will reduce the chance of introducing new bugs by a significant amount.


### Time spent:
30 hours
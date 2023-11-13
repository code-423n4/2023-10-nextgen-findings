# Vulnerability Details
Dangerous usage of `block.timestamp`. `block.timestamp` can be manipulated by miners.

# Impact
Usually to an extent of few seconds on Ethereum, or generally few percent of the block time on any EVM-compatible PoW network. In some cases, that can lead to serious consequences, such as:

--- An attacker can exploit a gambling contract that uses block.timestamp as a source of randomness, by adjusting the timestamp to get a favorable outcome.
--- An attacker can manipulate a timed auction contract that uses block.timestamp to determine the end of the bidding period, by extending or shortening the time window.
--- An attacker can influence a contract that uses block.timestamp to calculate interest rates, fees, or penalties, by changing the amount of time elapsed.

# Proof Of Concept
Link:- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L28

Codebase:-
'''
 function getWord(uint256 id) private pure returns (string memory) {
        
        // array storing the words list
        string[100] memory wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange", 
        "Blueberry", "Boysenberry", "Breadfruit", "Brush Cherry", "Canary Melon", "Cantaloupe", "Carambola", "Casaba Melon", "Cherimoya", "Cherry", "Clementine", 
        "Cloudberry", "Coconut", "Cranberry", "Crenshaw Melon", "Cucumber", "Currant", "Curry Berry", "Custard Apple", "Damson Plum", "Date", "Dragonfruit", "Durian", 
        "Eggplant", "Elderberry", "Feijoa", "Finger Lime", "Fig", "Gooseberry", "Grapes", "Grapefruit", "Guava", "Honeydew Melon", "Huckleberry", "Italian Prune Plum", 
        "Jackfruit", "Java Plum", "Jujube", "Kaffir Lime", "Kiwi", "Kumquat", "Lemon", "Lime", "Loganberry", "Longan", "Loquat", "Lychee", "Mammee", "Mandarin", "Mango", 
        "Mangosteen", "Mulberry", "Nance", "Nectarine", "Noni", "Olive", "Orange", "Papaya", "Passion fruit", "Pawpaw", "Peach", "Pear", "Persimmon", "Pineapple", 
        "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma", 
        "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"];

        // returns a word based on index
    @>    if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
        }

'''

# Mitigation
To avoid these vulnerabilities, you should use a more secure source of time, such as an oracle service or a commit-reveal scheme.
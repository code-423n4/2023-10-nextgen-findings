### Problem 1 

In contract "RandomPool" in file XRandoms.sol below, random word "Acai" is twice as likely to be picked as any other word, because both "wordsList" at index "id" == 0 and "id" == 1 give "Acai". Therefore this list is not uniformly random

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol

```
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
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
        }
```
### Problem 2

Public function "returnHighestBid()" is inefficient in logic, as the for-loop is not required.

https://github.com/code-423n4/2023-10-nextgen/blob/71d055b623b0d027886f1799739b7f785b5bc7cd/smart-contracts/AuctionDemo.sol#L65

The function is given as below:
```
    function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```

But is more efficient without the for-loop

```
    function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        uint256 length = auctionInfoData[_tokenid].length;
        if (length > 0) {
            if (auctionInfoData[_tokenid][length - 1].status == true) {
                return auctionInfoData[_tokenid][length - 1].bid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```




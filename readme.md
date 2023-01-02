# Build & Run
```bash
cargo build --release
cp ./target/release/anagram ./ # Add the .exe suffix if you're on Windows
./anagram # Again, add the .exe if you're on Windows
```
The resources/ directory must be adjacent to the anagram executable. The resources/anagram file and the resources/wordlist file are the only files that need to be in the resources directory.

The implementation of the solution algorithm will use as many threads & cores that are available on the system.

# My Anagram Phrase Solution

### 1. Canonical Sort & Filter the Word List
Every word is sorted by its letters, this is how anagrams are detected. Each sorted word is used as a key in an associative container (hashmap) whose values are the original, un-sorted word, EG:
**Sorted word:**  
"inkns"  
**Original words:**  
"sink", "skin"  
**Stored in an associative container (hashmap):**  
"ikns": ["sink", "skin"]

Duplicate words are filtered out as they are being added to the associative container. There are many ways to do this, refer to the code for implementation details.

### 2. Further Filtering of the Word List
Words that have letters which cannot be found in "Poultry Outwits Ants" are filtered out. This includes words with ' apostrophes.

Words that contain more letters than what are found in "Poultry Outwits Ants" are also excluded, an example might be "Apple" where there are 2 of the letter "P" present and only 1 of the letter "P" found in the original anagram phrase.

This level of filtering brings the total possible words from 99,175 down to 1,179.


### TODO: The above is good, cleanup the bottom
### .3 
"poultry outwits ants" is sorted and stored in its own container; a hashmap. The hashmap keys are the letters (excluding spaces) and the values are the quantity of the letter, EG:  
A:1  
I:1  
L:1  
N:1  
O:2  
P:1  
R:1  
S:2  
T:4  
U:2  
W:1  
Y:1  



3. The wordlist is filtered, only words that contain characters found in "poultry outwits ants" are preserved. Additional filtering is done based on character count. The result of the filtering is then stored in a hashmap container, which is also used to ensure that only unique entries are stored - a property of the hashmap will eliminate duplicates by only allowing the storage of unique keys.  A total of 1179 sorted anagrams is kept.

4. A hashmap is used to store the sorted anagram word and an embedded hashmap stores the character count. EG:  
"inks":  
I:1  
N:1  
K:1  
S:1

5. A list of the sorted anagrams is stored in a vector, which is further sorted by number of letters first with a secondary sort alphabetically. The larger words are at the beginning of the vector, this allows of fewer negatives when searching for words that can fit within the searched phrase. The alphabetical sorting doesn't have a practical purpose - other than to make repeated searches statistically deterministic and easier to follow by a human being. EG:  
["ainoosstttu", "ainoosstttu", "inooprssttu", ...]

6. The vector of sorted anagrams is iterated through, with the letters of each subtracted from the quantity of letters being searched for. EG:  
"Poultry Outwits Ants" is stored in a HashMap, as seen in Step (2) above. The character count of the sorted anagram words are then repeatedly subtracted: "ainoosstttu".

| Poultry Outwits Ants | Sorted Word | Result      |
|:---------------------|:------------|:-----------:|
|         A:1          |    A:1      |             |  
|         I:1          |    I:1      |             |  
|         L:1          |             |   L:1       |  
|         N:1          |    N:1      |             |  
|         O:2          |    O:2      |             |  
|         P:1          |             |   P:1       |  
|         R:1          |             |   R:1       |  
|         S:2          |    S:2      |             |  
|         T:4          |    T:3      |   T:1       |  
|         U:2          |    U:1      |   U:1       |  
|         W:1          |             |   W:1       |  
|         Y:1          |             |   Y:1       |

This subtraction is repeatedly done until the "Poultry Outwits Ants" character count reaches 0 and the length of the HashMap is then also 0. If the sorted word contains more characters than what is remaining, then the next word in the sorted word dictionary is selected.

7. A series of sorted words is then present in the phrase, a permutation algorithm is applied to the list of sorted words to determine all possible arrangements of the sorted word phrase.

8. The sorted words in a phrase is gibberish and meaningless, however the sorted words are keys to a vector of valid words, as seen in Step (3):  
"ikns":["sink", "skin"]  
These words are retrieved and recursively cycled through to find all possible word combinations within the phrase, with a space put in between each word to create a valid phrase.  

9. Finally, when a phrase composed of words is given, the permutation algorithm is applied again to find all possible permutations with the set of valid words. Note that duplicate words are not given special treatment; although it could be an optimization to handle duplicate words differently, it would explode the complexity of the permutation logic.  
Once a valid phrase is found, it's sent to the MD5 checksum comparator to determine if this phrase matches the given list of MD5 checksums.

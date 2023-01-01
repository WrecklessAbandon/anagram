# Build & Run
```bash
cargo build --release
cp ./target/release/anagram ./ # Add the .exe suffix if you're on Windows
./anagram # Again, add the .exe if you're on Windows
```
The resources/ directory must be adjacent to the anagram executable. The resources/anagram file and the resources/wordlist file are the only files that need to be in the resources directory.

# My Anagram Phrase Solution

1. Every word is sorted by its letters, this is how anagrams are detected. The sorted word is placed as a key in an associative container with the values as a list of the unsorted words, EG:  
"ikns":["sink", "skin"]  

2. "poultry outwits ants" is sorted and stored in its own container; a hashmap. The hashmap keys are the letters without spaces and the values are the quantity of the letter, EG:  
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

3. A hash map is used to detect and filter out duplicate entries. Another hashmap is used to store the sorted anagram word and an embedded hashmap stores the character count. EG:  
"inks":  
I:1  
N:1  
K:1  
S:1

4. A list of the sorted anagrams is stored in a vector, which is further sorted by number of letters first with a secondary sort alphabetically. The larger words are at the beginning of the vector, this allows of fewer negatives when searching for words that can fit within the searched phrase. The alphabetical sorting doesn't have a practical purpose - other than to make repeated searches statistically deterministic and easier to follow by a human being. EG:  
["ainoosstttu", "ainoosstttu", "inooprssttu", ...]

5. The vector of sorted anagrams is iterated through, with the letters of each subtracted from the quantity of letters being searched for. EG:  
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

6. A series of sorted words is then present in the phrase, a permutation algorithm is applied to the list of sorted words to determine all possible arrangements of the sorted word phrase.

7. The sorted words in a phrase is gibberish and meaningless, however the sorted words are keys to a vector of valid words, as seen in Step (3):  
"ikns":["sink", "skin"]  
These words are retrieved and recursively cycled through to find all possible word combinations within the phrase, with a space put in between each word to create a valid phrase.  

8. Finally, when a phrase composed of words is given, the permutation algorithm is applied again to find all possible permutations with the set of valid words. Note that duplicate words are not given special treatment; although it could be an optimization to handle duplicate words differently, it would explode the complexity of the permutation logic.  
Once a valid phrase is found, it's sent to the MD5 checksum comparator to determine if this phrase matches the given list of MD5 checksums.

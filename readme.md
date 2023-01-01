# Build & Run
```bash
cargo build --release
cp ./target/release/anagram ./ # Add the .exe suffix if you're on Windows
./anagram # Again, add the .exe if you're on Windows
```
The resources/ directory must be adjacent to the anagram executable. The resources/anagram file and the resources/wordlist file are the only files that need to be in the resources directory.

For the sake of brevity, I'll refer to "virtual cores" and "physical cores" as a processor. The applied solution will fully utilize a multi-core system. That is, if you have 32 processors available then all 32 processors will be used to find the solution.

In my experiment, I used a Zen 3 5950x and utlized all 16 physical cores plus an additional 16 threads via simultaneous multithreading (SMT) for a total of 32 processors. The hard and easy solutions take less than 100 milliseconds to compute and the medium difficulty solution takes approximately 83 seconds to compute. _The more processors that are available then the faster that the solution can be computed due to the multithreading implementation of this algorithm._ The algorithm can be forced to use a single thread but the default is to use all processors available.


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

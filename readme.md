# Build & Run
```bash
cargo build --release
cp ./target/release/anagram ./ # Add the .exe suffix if you're on Windows
./anagram # Again, add the .exe if you're on Windows
```
The resources/ directory must be adjacent to the anagram executable. The resources/anagram file and the resources/wordlist file are the only files that need to be in the resources directory.

The implementation of the solution algorithm will use as many threads & cores that are available on the system.

# My Anagram Phrase Solution

### 1. Canonical Sort & Initial Filtering of the Word List
Every word is sorted by its letters, this is how anagrams are detected. Each sorted word is used as a key in an associative container (hashmap) whose values are the original, un-sorted word, EG:
**Original words:**  
"sink", "skin"  
**Sorted words:**  
"ikns"  
**Stored in an associative container (hashmap):**  
"ikns": ["sink", "skin"]

Duplicate words are filtered out as they are being added to the associative container. There are many ways to do this, refer to the code for implementation details.

### 2. Further Filtering the Word List
Words that have letters which cannot be found in "Poultry Outwits Ants" are filtered out. This includes words with ' apostrophes.

Words that contain more letters than what are found in "Poultry Outwits Ants" are also excluded, an example might be "Apple" where there are 2 of the letter 'P' present and only 1 of the letter 'P' found in the original anagram phrase.

In the above example, the sorted anagram "inks" will be filtered out of the candidates of words since the letter 'k' does not appear in the list of letters of the anagram phrase.

This level of filtering brings the total possible words from 99,175 down to 1,179. The sorted words and associated words are then stored in a new container which is now the filtered word list.

### 3. 
"Poultry Outwits Ants" is sorted and stored in its own container; a hashmap. The hashmap keys are the letters (excluding spaces) and the values are the quantity of the letter, EG:  
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

Each sorted word is also stored as the same, EG:  
"ainoosstttu":["outstations"]  
A:1  
L:1  
O:2  
P:1  
R:1  
T:1  
U:1  
W:1  
Y:1  

Each sorted word is then looped over the "Poultry Outwits Ants" anagram phrase and the letters from the sorted word list are subtracted:

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

A record of the subtracted sorted word is kept:  
**"ainoosstttu":["outstations"]**  
It's possible that the same word can be subtracted multiple times, at that point the sorted word is recorded for each instance that is subtracted successfully.

The above loop is performed until the the result contains 0 letters remaining or until the word list has been exhausted.

### 4. Testing the Permutations
Once the remaining letters have reached exactly 0 instances, the words associated with each sorted word is then tested against the available MD5 checksums.

For example, the sorted anagram word "osttu" has 2 words associated with it: "touts" and "stout". Both words must be tested in the anagram phrase checksum, EG:  
"printout touts yawls" and "printout stout yawls" must both be tested. The permutation of each word must be tested within the phrase.

Another permutation must also be tested; the position of the word within the phrase. "printout touts yawls" has the following positional permutations:  
printout yawls stout  
stout printout yawls  
yawls stout printout  
stout yawls printout  
yawls printout stout  
stout yawls printout  
printout stout yawls  

There are 6 positional permutations from these 3 words. In addition to the positional permutations, the word permutations need to be substituted as well; meaning that "stout" can be replaced with "touts" and all positional permutations can be re-tested, bringing the total testing up to 12.

### 5. Summary of the solution:
Step 1 of the algorithm describes how to cononical sort and test for anagram words, duplicate words are also filtered out.

Step 2 of the algorithm describes the further filtering out of words from the word list by checking for existent characters found in "Poultry Outwits Ants", reducing the total word list to 1,179 words.

Step 3 of the alrogithm describes the basic looping that occurs by continuous subtraction of the number of letter instances within each word from the wordlist against the remaining number of letter instances found in "Poultry Outwits Ants".

Step 4 of the algorithm describes the testing of the MD5 checksums against all possible word permutations within a valid phrase as well as all positional permutations of the words within the phrase.

In summary, the algorithm is a filter, a subtraction of letter instances, permutations of an anagram word, permutations of each word within an anagram phrase, which is then finally tested against a given set of MD5 checksums.

### TODO Describe: A Note on a O(N = N - 1) Optimization
The following is an optimization for O(N = N - 1) looping. That is to say, the same word will be subtracted until it's fully exhausted from the list of anagram phrase possibilities.

In the case of "Outstations", it will be subtracted once and then a new word will be subtracted repeatedly until there all letters have been subtracted in "Poultry Outwits Ants".


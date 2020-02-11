# Rabin-Karp Algorithm


Used to find a substring of a string, i.e. finding `nan` in `banana`.

General idea:
1. Calculate the hash value for `nan`, `hash1 = n * 26**2 + a * 26 + n`.

**NOTE: Need to mod the hash value with a large prime number, then the hash value won't go too large.**

2. From the beginning of string `banana`, calculate the hash value for 3 chars, `ban`, `hash2 = b * 26**2 + a * 26 + n`.
3. `hash1 != hash2`, then slide the window forward to calculate the hash value for `ana`. It will not calculate for the whole string `ana` this time, it uses the rolling hashing, that is, `hash3 = hash(ana) = (hash2 - b * 26**2) * 26 + a`.
4. Repeat #3 until finding the substring or reaching the end of string.  

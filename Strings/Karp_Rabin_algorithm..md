# Rabin-Karp String Matching Algorithm in C++

This implementation uses a **polynomial rolling hash** to efficiently find occurrences of a pattern in a text.

---

## Intuition & Steps

- We convert each substring of length `m` in the text into a numeric hash value using the formula:

  \[
  H(s) = \sum_{i=0}^{m-1} (s[i] - 'a' + 1) \times prime^i \mod mod
  \]

- To avoid recomputing hash for every substring from scratch (which would be expensive), we use a **rolling hash** technique:
  1. Remove the contribution of the **leftmost character** of the previous substring.
  2. Divide the hash by `prime` to effectively shift powers down by one.
  3. Add the contribution of the **new rightmost character** multiplied by `prime^{m-1}`.
  
- Since division modulo a prime is non-trivial, we calculate the **modular inverse** of `prime` using Fermat's Little Theorem.

- When hashes match, we confirm by comparing the actual substring to avoid false positives due to collisions.

### Time complexity := best O(n+m) high probability and in worst case O(n*m) due to high collisions
---

## Code

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const ll prime = 31;              // Base value for polynomial rolling hash
const ll mod = 1000000007;        // Large prime for modulo to reduce collisions

// Modular exponentiation: computes (a^b) % m efficiently
ll power(ll a, ll b, ll m) {
    ll result = 1;
    a %= m;
    while (b > 0) {
        if (b & 1)                      // If b is odd
            result = (result * a) % m; // Multiply result with current base
        a = (a * a) % m;               // Square the base
        b >>= 1;                       // Divide exponent by 2
    }
    return result;
}

// Compute modular inverse using Fermat's Little Theorem:
// a^(-1) mod m = a^(m-2) mod m (valid only when m is prime)
ll modInverse(ll a, ll m) {
    return power(a, m - 2, m);
}

// Compute polynomial rolling hash of a string
// hash(s) = (s[0]*prime^0 + s[1]*prime^1 + ... + s[n-1]*prime^(n-1)) % mod
ll gethash(const string &s) {
    ll hash = 0;
    for (int i = 0; i < s.size(); i++) {
        hash = (hash + (s[i] - 'a' + 1) * power(prime, i, mod)) % mod;
    }
    return hash;
}

int main() {
    string text, pattern;
    cin >> text >> pattern;

    int n = text.size(), m = pattern.size();
    if (m > n) {
        cout << "Pattern is longer than text\n";
        return 0;
    }

    // Hash of the pattern and the first window of text
    ll patternHash = gethash(pattern);
    ll textHash = gethash(text.substr(0, m));

    // Precompute:
    ll primeInverse = modInverse(prime, mod);      // Modular inverse of prime
    ll highestPower = power(prime, m - 1, mod);   // prime^(m-1), used to add new char at end of window

    // Slide over text to find pattern matches
    for (int i = 0; i <= n - m; i++) {
        // If hash matches, confirm by comparing actual substring (to handle collisions)
        if (textHash == patternHash && text.substr(i, m) == pattern) {
            cout << "found at " << i << endl;
        }

        if (i < n - m) {
            // Remove leftmost char's contribution (multiplied by prime^0 = 1)
            textHash = (textHash - (text[i] - 'a' + 1) + mod) % mod;

            // Divide hash by prime to shift powers down by 1 (modular division)
            textHash = (textHash * primeInverse) % mod;

            // Add new char's contribution multiplied by prime^(m-1)
            textHash = (textHash + (text[i + m] - 'a' + 1) * highestPower) % mod;
        }
    }

    return 0;
}

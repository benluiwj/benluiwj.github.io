# CSES - Two Sets
I recently began attempting [CSES](https://cses.fi/) problems. More specifically, I am trying to complete this [list](https://cses.fi/alon/list/) 
of questions. Hopefully after completing these problems, I'll be able to solve more interesting problems while learning alot of new things too.
Without further ado, let's dive right in! 

# Question Statement
The full question can be found [here](https://cses.fi/alon/task/1092/). I have linked it below as well, feel free to attempt it too :-)

Your task is to divide the numbers from 1 to `n` into two sets of equal sum.
##### Input
The only input line contains an integer n.
##### Output
Print "YES", if the division is possible, and "NO" otherwise.
After this, if the division is possible, print an example of how to create the sets. First, print the number of elements in the first set followed by the elements themselves in a separate line, and then, print the second set in a similar way.
##### Constraints
`1 <= n <= 10^6`
###### Example 1
```
Input:
7

Output:
YES
4
1 2 4 7
3
3 5 6
```
###### Example 2
```
Input:
6

Output:
NO
```
## Intuition
To find whether it can be partitioned, we just check whether the sum of all the numbers from 1 to `n` is divisible by 2. A quick way to do this is by calculating the value of $\frac{n(n+1)}{2}$ 
#### Naive
DFS to find the partitions. This method is too slow even when $n <= 200$
#### Naive++?
The better way will be to use dp to find the pairs. Let the number of rows represent the values of target (from 0 to target) and the number of columns to represent the value of n (from 0 to n). Suppose there are `i` rows and `j` columns. Each `dp[i][j]` contains an `unordered_map` that contains the elements in the bucket. 

To populate the cell we use the following logic.
```c++
for (int i = 1; i < target + 1; i++) {
    for (int j = 1; j < n + 1; j++) {
      if (i == j)
        dp[i][j].insert(i);
      else if (i - j >= 0 && dp[i - j][j - 1].size() != 0) {
        dp[i][j].insert(j);
        for (auto k = dp[i - j][j - 1].begin(); k != dp[i - j][j - 1].end();
             k++) {
          dp[i][j].insert(*k);
        }
      } else {
        for (auto k = dp[i][j - 1].begin(); k != dp[i][j - 1].end(); k++) {
          dp[i][j].insert(*k);
        }
      }
    }
  }
```
In other words, the conditions are:
1. If `i == j` then we can just insert it and move on. Since we already hit the target
2. If `i - j >= 0` means that the value of the target does not exceed the value that we are currently trying to add and `dp[i-j][j-1].size() != 0` means that we attempt to re-use the result stored in `dp[i-j][j-1]` to populate our current cell. 
3. If all else fails, we just propagate the value from the previous cell.
All seems good here and I guess for typical coding interviews this is sufficient. However, this fails for the constraints of the current problem. 
### Solution
The solution uses some math. Basically the idea is that if we are able to divide the set into 2 different sets of equal sum, `n` must be of the form `4k` or `4k+3`. 
Case 1 (`n = 4k`):
For this case, we can put the first and last $\frac{n}{4}$ numbers into 1 bucket and the remaining numbers into the remaining bucket and we are done. 
Case 2 (`n = 4k + 3`):
Then we will have `4k, 4k+1, 4k+2, 4k+3`. Notice that `4k + 4k+3 == 4k + 1 + 4k + 2`. So if we recurse on these 2 cases, we will be able to get the 2 buckets accordingly. 
## Closing Thoughts
I was able to get the Naive and DP solution but what surprised me was that I had a memory issue because the size of the table I was trying to allocate was too big. Hence, I went to google how this should be done instead. It was quite interesting to see how math can make the problem and solution much simpler than a DP/DFS form. 
## Final Solution
```
#include <cmath>
#include <iostream>
#include <string>
#include <unordered_set>
#include <vector>

#define ll long long

using namespace std;

int main() {
  ll n;
  ll target = 0;

  cin >> n;
  ll total = (n * (n + 1)) / 2;
  if (total % 2) {
    cout << "NO" << endl;
    return 0;
  } else {
    cout << "YES" << endl;
    target = total / 2;
  }

  if (n % 4 == 0) {
    cout << n / 2 << endl;
    for (int i = 1; i <= n / 4; i++) {
      cout << i << " ";
    }

    for (int i = (3 * n) / 4 + 1; i <= n; i++) {
      cout << i << " ";
    }

    cout << endl;
    cout << n / 2 << endl;
    for (int i = n / 4 + 1; i <= (3 * n) / 4; i++) {
      cout << i << " ";
    }

    cout << endl;
    return 0;
  }

  unordered_set<ll> b1;
  unordered_set<ll> b2;

  ll value = (n - 3) / 4;
  while (value >= 0) {
    ll k3 = 4 * value + 3;
    ll k2 = 4 * value + 2;
    ll k1 = 4 * value + 1;
    ll k = 4 * value;

    b1.insert(k3);
    b1.insert(k);
    b2.insert(k1);
    b2.insert(k2);

    value--;
  }

  b1.erase(0);

  cout << b1.size() << endl;
  for (auto a : b1) {
    cout << a << " ";
  }
  cout << endl;
  cout << b2.size() << endl;
  for (auto a : b2) {
    cout << a << " ";
  }
  cout << endl;
  return 0;
}
```

## References
Hint was found [here](https://codeforces.com/blog/entry/86912)
Question can be found [here](https://cses.fi/alon/task/1092/)

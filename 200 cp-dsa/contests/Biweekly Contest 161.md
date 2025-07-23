**Title :** [Split Array by Prime Indices](https://leetcode.com/problems/split-array-by-prime-indices/)
**Tags** : #array, #math, #number-theory

#### Code
```cpp
class Solution {

public:

    static const int lim = 1e5;

    vector<int> sieve = vector<int>(lim, 1);

  

    void precompute(){

        sieve[0] = 0;

        sieve[1] = 0;

  

        for(int i = 2; i * i < lim; i++){

            if(sieve[i]){

                for(int j = i * i; j < lim; j += i){

                    sieve[j] = 0;

                }

            }

        }

    }

  

    long long splitArray(vector<int>& nums) {

        precompute();

  

        long long aSum = 0, bSum = 0;

  

        for(int i = 0; i < nums.size(); i++){

            if(sieve[i]) aSum += nums[i];

            else bSum += nums[i];

        }

  

        return abs(aSum - bSum);

    }

};

```
#### Logic
Just precompute the sieve, and then once you know whether an index is prime or not, take its sum accordingly, that is it
#### Notes
Nothing much, just put sieve in templates


**Title :** [Count Islands With Total Value Divisible by K](https://leetcode.com/problems/count-islands-with-total-value-divisible-by-k/)
**Tags** :  #array, #dfs, #bfs, #dsu, #matrix

#### Code
```cpp
class Solution {

public:

    void dfs(vector<vector<int>>& grid, int i, int j, long long& sum){

        if(i < 0 || i >= grid.size() || j < 0 || j >= grid[0].size() || grid[i][j] == 0) return;

  

        sum += grid[i][j];

        grid[i][j] = 0;

  

        dfs(grid, i + 1, j, sum);

        dfs(grid, i, j + 1, sum);

        dfs(grid, i - 1, j, sum);

        dfs(grid, i, j - 1, sum);

    }

    int countIslands(vector<vector<int>>& grid, int k) {

        int cnt = 0;

  

        for(int i = 0; i < grid.size(); i++){

            for(int j = 0; j < grid[0].size(); j++){

                if(grid[i][j]){

                    long long sum = 0;

                    dfs(grid, i, j, sum);

                    if(sum % k == 0) cnt++;

                }

            }

        }

  

        return cnt;

    }

};
```
#### Logic
Pretty easy question, this too, js
#### Notes









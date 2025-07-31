**Title :** [Maximum Median Sum of Subsequences of Size 3](https://leetcode.com/problems/maximum-median-sum-of-subsequences-of-size-3/)
**Tags** : #math, #implementation #greedy 

#### Code
```cpp
class Solution {

public:

    long long maximumMedianSum(vector<int>& nums) {

        int n = nums.size();

        long long sum = 0;
        sort(nums.begin(), nums.end());
        for(int i = n - 2; i >= n / 3; i-=2) sum += nums[i];

        return sum;

    }

};
```
#### Logic
So, basically you can select any three elements you want right, and you want to maximize the median of the sums. so basically try to get the maximum medians.
now, in three numbers the median would be the second biggest number. so what you do is first sort the array, and then the last element would be the biggest, so you cannot have this as the median, so take the second largest element, now after you do this, pick the next second largest element, what we are doing here is essentially, 
smaller number, second larger number, largest number, so this, way we try to maximize our median taking thing
> 1, 2, 3, 4, 5
> take 4 first, then i-=2 you come to three, and then that is it, run the loop until you are able to select three elements only, and this is it, btw, you do this because, you remove the three elements once you do this, you cannot use them again, else this question would be even easier i question, then you would do -1 instead of 2

#### Notes
Nothing much, pretty good question, for like fundamentals, i was thinking of something like heap here, but that was pointless, wait till you get the tags of this thing.



**Title :** [Maximum Number of Subsequences After One Inserting](https://leetcode.com/problems/maximum-number-of-subsequences-after-one-inserting/)
**Tags** : #string, #greedy, #implementation 

#### Code

#### Logic
So, yeah this is a pretty good question, you may think this is dp, but the constraints, do not fit unless you can think of a 1d dp, which seems impossible, at least for me.
so now you have to put insert either l, c or t, such that the total occurrences of 'lct' is maximized.
so, now obv, if you want to put l, you obv insert it in the first place right, then you get more 'lct's, and if you want to insert a 't' the most optimal place to insert a 't' would be at the last, this makes sense right.
now to put a 'c' you have to make sure that, you put it in a place such that, there are a lot of 'l's before it and a lot of 't's after this. this you find using prefix and suffix arrays.
now basically find in which case you get the most 'lct's, and return that.
#### Notes


**Title : **[Minimum Jumps to Reach End via Prime Teleportation](https://leetcode.com/problems/minimum-jumps-to-reach-end-via-prime-teleportation/)
**Tags** :  #bfs #number-theory #shortest-path 

#### Code:
```cpp
int const MAXNUM = 1e6 + 5;

int spf[MAXNUM + 1] = {0};

void precompute()

{ // spf[0] and spf[1] are undefined;

    for (int i = 2; i <= MAXNUM; i++)

    { // nlogn sieve;

        if (!spf[i]) for (int j = i; j <= MAXNUM; j += i) spf[j] = i;

    } // is spf[i] = 0, means its prime, make every j = i;

}

  

class Solution {

public:
    int const MOD = 1e9 + 7;
    int minJumps(vector<int>& nums)
    {
        if (!spf[2]) precompute();

        int const n = nums.size();

        unordered_map<int, vector<int>> idx; // idx[p] = indices divisible by prime p;

        for (int i = 0; i <= n - 1; i++)

        { // insert index in all prime factors;

            int x = nums[i];

            while (spf[x])

            {

                idx[spf[x]].push_back(i);

                x /= spf[x];

            }

        }

  

        int di[2] = {-1, 1};

        vector<int> dist(n, MOD);

        dist[0] = 0;

  

        queue<int> to_visit;

        to_visit.push(0);

  

        while (to_visit.size())

        {

            int i = to_visit.front();

            to_visit.pop();

  

            for (int d = 0; d <= 1; d++)

            {

                int ni = i + di[d];

                if (ni >= 0 && ni <= n - 1 && dist[i] + 1 < dist[ni])

                {

                    dist[ni] = 1 + dist[i];

                    to_visit.push(ni);

                }

            }

            if (spf[nums[i]] != nums[i]) continue;

            while (idx[nums[i]].size())

            {

                int ni = idx[nums[i]].back();

                if (dist[i] + 1 < dist[ni])

                {

                    dist[ni] = 1 + dist[i];

                    to_visit.push(ni);

                }

                idx[nums[i]].pop_back();

            }

        }

  

        return dist[n - 1];

    }

};
```
#### Logic
so, the first hint that you have is you need to find the minimum number of steps to reach some destination. one approach could be to use dynamic programming, but apparently the transition and states do not make sense. and another thing you can use to find the minimum steps would be to use bfs, so basically that would be to do graph modelling. so now the nodes are given in the nums array, now you have to connect edges, now connecting the edges of everything would be bad, cause there would be too many edges, which would lead to tle. so instead of connecting the edges, we would just add the edges into the q while doing the bfs, simulation that there is an edge between the nodes.
so now the crux of the question is that, from any index you can move to index i - 1, or 1 + 1, and if nums[i] is a prime number p, you have one additional point to jump to, basically any index j such that nums[j] % p == 0, so basically nums[j] should be a multiple of p. so now we have to find all the factors of a composite number, to easily be able to navigate as per the question, now comes the implementation, which is somewhat the harder part, have to get better at doing this.

alright, so we use spf array to find the smallest prime factor of a number, cause using this we would be able to factorize any number in basically root n, i guess.
so, now to check all the numbers we can jump to from number p we need to precompute all the indices we can jump to, now to find this we will use the spf array,
so now go through the nums array, and then pick the numbers, if its a prime, 
#### Notes














**Title :** [Maximum Median Sum of Subsequences of Size 3](https://leetcode.com/problems/maximum-median-sum-of-subsequences-of-size-3/)
**Tags** : #math, #implementation 

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
now, in three numbers the median would be the second biggest number. now i can think of two approach to this question, one would be to sort it, and once you do this, the last element would be the biggest right, and the second last element would be a candidate for you median right. 
#### Notes





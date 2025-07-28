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
now basically find the 
#### Notes










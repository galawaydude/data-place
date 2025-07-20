**Title :** [Check Divisibility by Digit Sum and Product](https://leetcode.com/problems/check-divisibility-by-digit-sum-and-product/)
**Tags** : #implementation, #simulation

#### Code
```cpp
class Solution {

public:

    bool checkDivisibility(int n) {

        int temp = n;

        int prod = 1, sum = 0;

        while(n > 0){

            sum += n % 10;

            n /= 10;

        }

  

        n = temp;

        while(n > 0){

            prod *= n % 10;

            n /= 10;

        }

  

        // cout << sum << " " << prod;

        return (temp % (sum + prod) == 0);

    }

};
```
#### Logic
Nothing much here
#### Notes
Nothing much here


**Title :** [Count Number of Trapezoids I](https://leetcode.com/problems/count-number-of-trapezoids-i/) 
**Tags** :  #geometry, #implementation, #map
#sr

#### Code
```cpp
class Solution {

public:

    int const mod = 1e9 + 7;

    int countTrapezoids(vector<vector<int>>& points) {

        map<int, long long> mp;
        for(auto& it: points) mp[it[1]]++;
        long long lines = 0, cnt = 0;

        for(auto& [k, v]: mp){

            int numLines = (v * (v - 1)) / 2;
            cnt = (cnt + numLines * lines) % mod;
            lines = (lines + numLines) % mod;
        }
        return (int)cnt;

    }

};
```
#### Logic
See, i was close, while solving this problem, just missed some very important points, i was right to use combinations, and even the `c(n, 2)` thing, but what i did miss, was how to calculate this thing, we are basically doing this step by step.

> convex shape is basically the one where all the interior angles are less than 180, and any line segment between two points, inside the shape lies entirely withing the shape.

So, we need to horizontal lines that are parallel to each other, now this is basic common sense that, two horizontal lines are always parallel to each other, as long as they do not overlap with each other.

so, now to get one horizontal line, the condition is that, the y co-ordinate of both the points, should be same, now this is where the math comes into play here, assume you have 3 points, with the same y co-ordinates, you can make a horizontal line between the first two, second two, and the last two, this is basically the combination formula, now for every horizontal line from this y coordinate, you need another horizontal line from another y coordinate, to make a horizontal trapezoid, so, basically get the total number of lines from each y co-ordinate and multiply them, and you get all the possible combinations.

so, you need to make a horizontal 
#### Notes
Nothing much here










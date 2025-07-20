**Title :** [Check Divisibility by Digit Sum and Product](https://leetcode.com/problems/check-divisibility-by-digit-sum-and-product/)
**Tags** : #implementation, #simulation

#### Code
```
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
**Tags** : 

#### Code
#### Logic
#### Notes









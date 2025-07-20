**Title** : [Check Divisibility by Digit Sum and Product](https://leetcode.com/problems/check-divisibility-by-digit-sum-and-product/)
**Tags** : #simulation, #implementation
#### Logic 
Nothing much to do here, very basic code

#### Notes
Nothing much here either

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







**Title** : 
**Tags** :

#### Logic


#### Notes


#### Code




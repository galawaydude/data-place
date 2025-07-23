**Title : **[Process String with Special Operations I](https://leetcode.com/problems/process-string-with-special-operations-i/)
**Tags** : #string, #simulation 

#### Code
```cpp
class Solution {

public:

    string processStr(string s) {

        string ans = "";

        for(auto& it: s){

            if(islower(it)) ans+=it;

            if(it == '*' && !ans.empty()) ans.pop_back();

            if(it == '#'){

                string temp = ans;

                ans.append(temp);

            }

            if(it == '%'){

                reverse(ans.begin(), ans.end());

            }

        }

  

        return ans;

    }

};
```
#### Logic
Nothing much, god knows why its 
#### Notes





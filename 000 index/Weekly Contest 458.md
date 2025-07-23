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
Nothing much, god knows why its medium, just basic simulation.
#### Notes
Nothing Much



**Title :** [Minimize Maximum Component Cost](https://leetcode.com/problems/minimize-maximum-component-cost/)
**Tags** : #binary-search, #sort, #dsu, #graph 

#### Code
```cpp
// obv dsu

// i remember doing such a question, for each component also store the max edge weight

// now you need to minimize this, so i guess once you get the above thing, start removing the max weight edge, and keep on doing this, until you have the number of componenets that you need.

// i guess this should work

// or i guess instead of breaking after i create the components, i make the compoenents, after taking the smallest one to begin with, this way i do not have to write a code, to remove elements from compoenents

class DisjointSets {

  private:

    vector<int> parents;

    vector<int> sizes;

  

  public:

    DisjointSets(int size) : parents(size), sizes(size, 1) {

        for (int i = 0; i < size; i++) { parents[i] = i; }

    }

  

    int find(int x) { return parents[x] == x ? x : (parents[x] = find(parents[x])); }

    bool unite(int x, int y) {

        int x_root = find(x);

        int y_root = find(y);

        if (x_root == y_root) { return false; }

  

        if (sizes[x_root] < sizes[y_root]) { swap(x_root, y_root); }

        sizes[x_root] += sizes[y_root];

        parents[y_root] = x_root;

        return true;

    }

  

    bool connected(int x, int y) { return find(x) == find(y); }

  

    int size(int i) { return sizes[find(i)]; }

};

class Solution {

public:

    int minCost(int n, vector<vector<int>>& edges, int k) {

        int cnt = 0;

        sort(edges.begin(), edges.end(),[](auto& a, auto& b){

            return a[2] < b[2];});

  
  

        // for(auto& it: edges) cout << it[2] << " ";

        // cout << endl;

  

        DisjointSets dsu(n);

        for(auto& it: edges){

            int u = it[0], v = it[1];

  

            if(dsu.unite(u, v)){

                cnt++;

                if(cnt == n - k) return it[2];

            }

        }

  

        return 0;

    }

};
```
#### Logic
so, first and foremost thing, this question can be solved using binary search also, try to do that first. 
so, what i did was, since we wanted to minimize the edges, its better to take the smallest edges first right, so i sorted the edges first (the lamda expression took way too much time, properly learn how to make them alright).
once this was done, i making dsu, connecting all the edges, and finding out how many components i have so far, now i need atmost k connected components
#### Notes









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
so, what i did was, since we wanted to minimize the edges, its better to take the smallest edges first right, so i sorted the edges first (the lambda expression took way too much time, properly learn how to make them alright).  
once this was done, i making dsu. and joining nodes using their edges, since i want to minimize the edges, i want to take as little edges as possible right, cause then i would have the smallest biggest edge, now once i connect two things with my edge, i want to check how many components, so i basically need a relation between number of edges, which is stored in cnt, and the total nodes to get the number of components. see we are using Kruskal's, because this can be used to construct a **spanning forest**.
> also, just to make it clear, a **spanning forest** is basically an extension of a **spanning tree**, but for graphs that might not be fully connected. so like, if your graph has multiple disconnected parts (or you intentionally break it into multiple components like in this question), then you can’t build one single spanning tree. instead, you build one **spanning tree per component**, and the collection of all those trees is called a **spanning forest**.  
each tree covers all the nodes in its component and has no cycles, just like a normal spanning tree. so yeah, in this problem when we break the graph into `k` components, we’re actually building a spanning forest with `k` trees and exactly `n - k` edges.

once i have this cnt, which tells me how many edges i've added, i know that the number of components is just `n - cnt`. now the actual question says i can have **at most** `k` components. so if the number of components after building my forest (i.e., `n - cnt`) is `<= k`, that means i'm allowed to use this max edge weight i just added. but remember, we want to **minimize** the max cost among components, so that means i can binary search on the possible edge weights — for each guess (`mid`), i only allow edges with weight `<= mid` and try to build the forest. if the number of components i get is `<= k`, it’s a valid answer, so i move left in the binary search to try and minimize it even more. otherwise, if i can’t form enough components (i.e., i have too few), i go right.

so in the binary search loop:

- i pick `mid` (a weight guess),
    
- run kruskal using only edges with weight `<= mid`,
    
- count how many components are formed (`n - cnt`),
    
- check if it’s `<= k`.
    

this becomes my condition to adjust the binary search.

at the end, the smallest `mid` that gives `<= k` components is my answer, because it’s the minimal max edge weight allowed across the components.
#### Notes









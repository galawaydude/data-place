
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
Pretty easy question, this too, just find a cell, that has a value, and apply dfs from that cell, and then just get the sum of the complete region, and check the condition you have to.
#### Notes
Nothing much, just try to do this question using dsu, seems a good use case for dsu.


**Title :** [Network Recovery Pathways](https://leetcode.com/problems/network-recovery-pathways/)
**Tags** : #array, #binary-search, #dp, #graph, #topo-sort, #heap, #shortest-path

#### Code
```cpp
class Solution {

public:

    long long const inf = 1e18;

    vector<vector<pair<int, long long>>> adj;

    vector<vector<int>> graph;

    vector<int> topo;

    vector<bool> online;

    unsigned lonk;

  

    void kahn() {

        int n = graph.size();

        vector<int> indeg(n);

  

        for (auto& it : graph)

            for (auto& it2 : it)

                indeg[it2]++;

  

        queue<int> q;

        for (int i = 0; i < n; i++)

            if (indeg[i] == 0) q.push(i);

  

        while (!q.empty()) {

            int node = q.front(); q.pop();

            topo.push_back(node);

            for (auto& it : graph[node]) {

                indeg[it]--;

                if (indeg[it] == 0) q.push(it);

            }

        }

    }
    bool isPossible(int x) {

        int n = adj.size();

        vector<long long> dist(n, inf);

        dist[0] = 0;

  

        for (int u : topo) {

            if (dist[u] == inf) continue;

            for (auto& [v, cost] : adj[u]) {

                if (cost < x) continue;

                if (!online[v] && v != n - 1) continue;

                dist[v] = min(dist[v], dist[u] + cost);

            }

        }
        return dist[n - 1] <= k;

    }
    int findMaxPathScore(vector<vector<int>>& edges, vector<bool>& _online, long long _k) {

        k = _k;

        online = _online;

        int n = online.size();

        adj.assign(n, {});

        graph.assign(n, {});

        int maxCost = 0;
        for(auto& it : edges) {

            int u = it[0], v = it[1], c = it[2];

            adj[u].push_back({v, c});

            graph[u].push_back(v);

            maxCost = max(maxCost, c);

        }
        kahn();
        if (!isPossible(0)) return -1;

        long long lo = 0, hi = maxCost;
        while (lo < hi) {
            int mid = lo + (hi - lo + 1) / 2;
            if (isPossible(mid)) lo = mid;
            else hi = mid - 1;

        }
        return lo;

    }

};
```
#### Logic
So, yeah pretty good question for me, incredibly good. so first thing, is return the maximum score, so basically we are trying to maximize something, and you should have learnt by now, that you need to try to apply binary search in this, binary search on answer. now path score is basically the edge weight, you have to find the maximum edge in a path, that would take you from the 0th node to the nth node. and one additional constraint you have to take care of is, you entire path cost should not exceed k, so we have to try to minimize this as much as possible, right, so pick the shortest path, with the lowest cost, this begs use to use Dijkstra, but since its a DAG, we can use topo sort, and process nodes in a particular order, and relax them accordingly, kinda doing a dp approach. so, yeah the basic step for doing bsoa would be to get the limits, the max and the min, and max would be the max edge in the graph, and the min would be 0.
you basically have to try to maximize your answer right. so pick a mid, and check if you are able to reach from the first node, to the last node, with this edge being your path score, this means that in the path you take, this weight should be the smallest, and if this is possible, you can try to increase it right, and if this is not possible, you have to decrease the weight that you picked. now how to know if its possible, in the check function what you do is, start from the first node, and using topo sort and dp, try to find the shortest path from the first node to the last node, such that, all the edges that you take, should be greater then or equal to the value you are checking in the check function. and at the end, if the cost to reach the last mode, is less then or equal to k, then return true, else return false;
(in this question we are asked to maximize the value, we can also be asked the opposite, asked to minimize the values, then if true,we explore the left part, not the right part)'
#### Notes
learnt that in almost all DAG questions, and dp approach can be taken, and topo sort is the goat.













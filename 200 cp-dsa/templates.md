### 1. dsu
```cpp
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
```

### 2. sieve
```cpp
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
```

### 3. Kahn's algorithm 
```

```
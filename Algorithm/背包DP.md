背包DP模版：
```cpp
  int numSquares(int n) {
        vector<int> f(n+1);
        f[1] = 1;
        //从第二个开始
        for(int i = 2;i <= n;i ++){
            //当前最优解
            int curMin = INT_MAX;
            for(int j = 1;j * j <= i;j ++){
                //寻找并记录最优解
                curMin = min(curMin,f[i - j*j]);
            }
            //加上此次执行的+1
            f[i] = 1 + curMin;
        }
        return f[n];
    }
```

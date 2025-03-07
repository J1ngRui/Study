回溯算法模板:
```cpp
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }
    for (选择 : 本层集合中的元素) {
        处理节点;
        backtracking(路径, 选择列表); // 递归
        撤销处理; // 回溯
    }
}
```
---------------------------------------------------------------------------------------
全排列（求解数组的全排列组合）  
[https://leetcode.cn/problems/permutations/description/?envType=study-plan-v2&envId=top-100-liked]  
```cpp
void backtrack(vector<int>& nums,int first,int len){
        //如果此时first == len,说明遍历到了结尾，所以直接记录下当前交换完成的nums
        if(first == len){
            res.emplace_back(nums);
            return;
        }
        //将从first开始进行遍历
        //这里的i实际上代表的是需要固定的元素（1开头的组合、2开头的组合……）
        for(int i = first;i < len;i++){
            //首次交换为自身交换
            swap(nums[i],nums[first]);
            //然后查看first开头的排列组合
            backtrack(nums,first+1,len);
            //恢复原状
            swap(nums[i],nums[first]);
        }
    }
```
---------------------------------------------------------------------------------------
求所有的子集问题：  
```cpp
vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> ans = {{}};
        int n = nums.size();
        for(int i = 0;i < n;i ++){
            int m = ans.size();
            for(int j = 0;j < m;j ++){
                ans[j].push_back(nums[i]);
                ans.push_back(ans[j]);
                ans[j].pop_back();
            }
        }
        return ans;
    }
```
在这里可以你注意到，回溯算法的核心在于完成某次遍历后的"恢复原状"

# 数组

## 两数之和

思路：

使用哈希表。

代码：

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> mp;

    for(int i=0;i<nums.size();i++){
        int t = target - nums[i];
        if(mp.count(t))
            return {mp[t],i};
        mp[nums[i]] = i;
    }

    return {};
}
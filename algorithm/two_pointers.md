# 双指针法

## 双指针法的用途：

双指针法通常有如下几个用途：

1.对于排序好的数组，我们可以一个左指针，一个右指针。nums[left] + nums[right] 大于目标值时右移left指针，小于左移right指针。

2.接水问题：判断板中容积。方法如下：每次选择移动最小的板，以最小的板作为高度。计算当前最小板能形成的最大容积，将maxnum返回。

3.判断柱子间能放雨水。方法：**变种双指针**先向左扫描，记录每个位置左方最大的高度、后向有扫描，记录每个位置最大的板。此时观察左右两个数组，形成双指针，左右两个数组分别指向构成该位置最大的两块板子。此时我们只需要去左右数组的最小值-当前高度即可计算出当前位置对储水量的贡献。

4.快慢指针：判断链表是否存在环。

## 15.三数之和 //16.最接近三数之和

题目：  
给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请你返回所有和为 0 且不重复的三元组。

题解：  
最经典的双指针变种问题。乍一看会让人感觉到需要三指针进行处理但实际上只需要对nums进行排序，之后我们按顺序指定一个数字now，然后对now后的数组进行双指针算法寻找对应的-now即可。
```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int n = nums.size();
        sort(nums.begin(), nums.end());
        vector<vector<int>> ans;
        // 枚举 a
        for (int first = 0; first < n; ++first) {
            // 需要和上一次枚举的数不相同
            if (first > 0 && nums[first] == nums[first - 1]) {
                continue;
            }
            // c 对应的指针初始指向数组的最右端
            int third = n - 1;
            int target = -nums[first];
            // 枚举 b
            for (int second = first + 1; second < n; ++second) {
                // 需要和上一次枚举的数不相同
                if (second > first + 1 && nums[second] == nums[second - 1]) {
                    continue;
                }
                // 需要保证 b 的指针在 c 的指针的左侧
                while (second < third && nums[second] + nums[third] > target) {
                    --third;
                }
                // 如果指针重合，随着 b 后续的增加
                // 就不会有满足 a+b+c=0 并且 b<c 的 c 了，可以退出循环
                if (second == third) {
                    break;
                }
                if (nums[second] + nums[third] == target) {
                    ans.push_back({nums[first], nums[second], nums[third]});
                }
            }
        }
        return ans;
    }
};
```

## 42接雨水

题目：
给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

题解：
参考题型3.记录每个位置左方最大的高度、后向有扫描，记录每个位置最大的板。此时观察左右两个数组，形成双指针，左右两个数组分别指向构成该位置最大的两块板子。此时我们只需要去左右数组的最小值-当前高度即可计算出当前位置对储水量的贡献。
```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        for (int now = 0; now <= nums.size() - 3; now++) {
            if(now>=1 && nums[now] == nums[now-1]){
                continue;
            }
            int left = now + 1;
            int right = nums.size() - 1;
            while (right > left) {
                int sum = nums[now] + nums[left] + nums[right];
                if (sum == 0 ) {
                    vector<int> temp = {nums[now], nums[left], nums[right]};
                    res.push_back(temp);
                    left++;
                    while (left < right && nums[left] == nums[left - 1]) {
                        left++;
                    }
                } else {
                    if (sum < 0) {
                        while (left < right && nums[left] == nums[left + 1]) {
                            left++;
                        }
                        left++;
                    } else {
                        while(right > left && nums[right]==nums[right-1]){
                            right--;
                        }
                        right--;
                    }
                }
            }
        }
        return res;
    }
};
```


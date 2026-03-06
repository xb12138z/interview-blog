# 哈希表

哈希表方面的题其实比较简单，大多数考验怎么灵活的运用哈希表降低时间复杂度。如果有难度的话一般是对字符串进行哈希记录时要记得使用滚动hash（hard）。或者衍生到KMP 、 扩展KMP 字符串匹配算法。

## 49.字母异位词分组【mid】

题目：给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

思路：

方法一：我们可以给每个单词进行排序，然后在哈希表中以排序好的值为键进行分组

代码：
```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> mp;
        for (string& str: strs) {
            string key = str;
            sort(key.begin(), key.end());
            mp[key].emplace_back(str);
        }
        vector<vector<string>> ans;
        for (auto it = mp.begin(); it != mp.end(); ++it) {
            ans.emplace_back(it->second);
        }
        return ans;
    }
};
```

方法二：自己进行哈希运算的一种方法，下方给出示例代码。这个是写了一个回调函数，但是有点抽象，可以看我给出的 214最短回文串 的题解

代码：
```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // 自定义对 array<int, 26> 类型的哈希函数
        auto arrayHash = [fn = hash<int>{}] (const array<int, 26>& arr) -> size_t {
            return accumulate(arr.begin(), arr.end(), 0u, [&](size_t acc, int num) {
                return (acc << 1) ^ fn(num);
            });
        };

        unordered_map<array<int, 26>, vector<string>, decltype(arrayHash)> mp(0, arrayHash);
        for (string& str: strs) {
            array<int, 26> counts{};
            int length = str.length();
            for (int i = 0; i < length; ++i) {
                counts[str[i] - 'a'] ++;
            }
            mp[counts].emplace_back(str);
        }
        vector<vector<string>> ans;
        for (auto it = mp.begin(); it != mp.end(); ++it) {
            ans.emplace_back(it->second);
        }
        return ans;
    }
};
```

## 214最短回文子串【hard】

题目：给定一个字符串 s，你可以通过在字符串前面添加字符将其转换为回文串。找到并返回可以用这种方式转换的最短回文串。

解法 ：

滚动哈希。根据哈希表的原理，我们目的是为了研制一套对应的算法用来进行 对表述的对象 到 一个近似唯一的映射。所以我们不妨将我们要处理的字符串想象成  一个大于26进制的数字，我们只需要计算出对应的值就可以获得唯一的对应表述的对象。

此题还有一个亮点 : 我们可以在一次遍历中计算正序和逆序的值。（int 情况下感觉会很好用）

代码：
```cpp
class Solution {
public:
    string shortestPalindrome(string s) {
        int n = s.size();
        int base = 131, mod = 1000000007;
        int left = 0, right = 0, mul = 1;
        int best = -1;
        for(int i = 0 ;i < n ;i++){
            left = ((long long)left * base + s[i]) % mod;
            right = ((long long)mul * s[i] +right) % mod;
            mul = (long long)mul *base % mod;
            if(left == right){
                best = i;
            }
        }
        string add = (best == n-1 ? "" : s.substr(best + 1, n));
        reverse(add.begin(),add.end());
        return add+s;
    }
};
```

## 2223构造字符串的总得分和【hard】

题目：你需要从空字符串开始 构造 一个长度为 n 的字符串 s ，构造的过程为每次给当前字符串 前面 添加 一个 字符。构造过程中得到的所有字符串编号为 1 到 n ，其中长度为 i 的字符串编号为 si 。比方说，s = "abaca" ，s1 == "a" ，s2 == "ca" ，s3 == "aca" 依次类推。si 的 得分 为 si 和 sn 的 最长公共前缀 的长度（注意 s == sn ）。给你最终的字符串 s ，请你返回每一个 si 的 得分之和 。

解法：

Z函数 —— 扩展KMP算法的标准样板题。（得到 当前字符串中 从任意位置开始与字符串开头的最长匹配长度数组）.本题灵神给的解法不是完全O（n）的，我按照标准的Z函数优化了一下。

```cpp

class Solution {
public:
    long long sumScores(string s) {
        int len = s.size();
        vector<int> res(len);
        long long ans = 0;
        for (int i = 1, l = 0, r = 0; i < len; i++) {//l、r分别为模板子串的左右边界
            if (i <= r && res[i - l] < r - i + 1) {//判断与模板子串的匹配情况，如果在匹配结果没有超过模板子串，则直接匹配成功，获得结果
                res[i] = res[i - l];
            } else {
                res[i] = max(r - i , 0);//当与模板子串完全匹配时，我们直接从模板子串下一位开始对比。 否则时间复杂度会变为O（n^2）
                while ((i + res[i]) < len && s[i + res[i]] == s[res[i]]) {//找到最长模板子串
                    res[i]++;
                }
            }
            if(i + res[i] - 1 > r){
                r = i + res[i] - 1;
                l = i;
            }
            ans += res[i];
        }
        return ans+len;
    }
};

```

## KMP算法

题目：



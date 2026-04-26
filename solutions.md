# 438. 找到字符串中所有字母异位词

## 题目描述

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

---

### 示例 1
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
plain
复制

**解释：**
- 起始索引等于 `0` 的子串是 `"cba"`，它是 `"abc"` 的异位词。
- 起始索引等于 `6` 的子串是 `"bac"`，它是 `"abc"` 的异位词。

### 示例 2
输入: s = "abab", p = "ab"
输出: [0,1,2]
plain
复制

**解释：**
- 起始索引等于 `0` 的子串是 `"ab"`，它是 `"ab"` 的异位词。
- 起始索引等于 `1` 的子串是 `"ba"`，它是 `"ab"` 的异位词。
- 起始索引等于 `2` 的子串是 `"ab"`，它是 `"ab"` 的异位词。

---

## 解题思路

### 错误尝试：暴力解法（使用 Set）

> ⚠️ **注意：此解法存在逻辑缺陷，仅作分析记录。**

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        length = len(p)
        ans = []
        for i in range(len(s) - length + 1):
            set_s = set(s[i:i+length])
            key = True
            for j in p:
                if j not in set_s:
                    key = False
            if key is True:
                ans.append(i)
        return ans
问题分析：
表格
问题点	说明
无法判断字符频次	set 只能反映出现了什么元素，不能反映元素出现了几次。
反例	若 p = "aa"，s = "abb"，使用 set 会误判为异位词。
正确解法：滑动窗口 + 哈希表
Python
from collections import Counter

class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        m, n = len(s), len(p)
        if m < n:
            return []
        
        ans = []
        cnt_p = Counter(p)          # 目标字符频次
        window = Counter(s[:n-1])   # 先存入前 n-1 个字符
        
        for i in range(n - 1, m):
            # 1. 右侧窗口扩张：加入新字符
            window[s[i]] += 1
            
            # 2. 判断是否匹配
            if window == cnt_p:
                ans.append(i - n + 1)
            
            # 3. 左侧窗口收缩：移除最左边字符
            left_char = s[i - n + 1]
            window[left_char] -= 1
            if window[left_char] == 0:
                del window[left_char]  # 及时删除 0 值键，保证字典等价性判断正确
        
        return ans
算法流程图解：
plain
复制
初始化窗口：[s0, s1, ..., s(n-2)]  (长度 n-1)

循环 i 从 n-1 到 m-1:
    ├─ 窗口右侧加入 s[i]        →  window[s[i]] += 1
    ├─ 比较 window 与 cnt_p
    │      ├─ 相等 → 记录起始索引 i-n+1
    │      └─ 不等 → 继续
    └─ 窗口左侧移除 s[i-n+1]    →  若频次归零则删除该键
复杂度分析
表格
指标	复杂度	说明
时间复杂度	O(m)	遍历字符串 s 一次，每次字典操作 O(1)。
空间复杂度	O(Σ)	哈希表存储字符频次，Σ 为字符集大小（本题最多 26 个小写字母）。
关键易错点
窗口初始化长度：先只放入 n-1 个字符，在循环中再放入第 n 个，避免边界处理混乱。
零值键清理：Counter 中值为 0 的键不会自动删除，必须手动 del，否则 window == cnt_p 会失败。
边界条件：记得处理 m < n 时直接返回空列表。

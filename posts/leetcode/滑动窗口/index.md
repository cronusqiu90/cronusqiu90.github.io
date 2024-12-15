# 算法之滑动窗口



### 定长滑动窗口

三部曲:
1. 进入窗口，更新统计量
2. 更新结果
3. 移出窗口，更新统计量

leetcode.1456 定长子串中元音的最大数目:
```text
给你字符串 s 和整数 k 。
请返回字符串 s 中长度为 k 的单个子字符串中可能包含的最大元音字母数。
英文中的 元音字母 为（a, e, i, o, u）。
```

```python
class Solution:
    def maxVowels(self, s: str, k: int) -&gt; int:
        ans = vowel = 0
        for i, c in enumerate(s):
            # 1. 进入窗口
            if c in &#34;aeiou&#34;:
                vowel &#43;= 1
            if i &lt; k - 1:  # 窗口大小不足 k
                continue
            # 2. 更新答案
            ans = max(ans, vowel)
            # 3. 离开窗口
            if s[i - k &#43; 1] in &#34;aeiou&#34;:
                vowel -= 1
        return ans
```

leetcode.2461  长度为 K 子数组中的最大和
```text
给你一个整数数组 nums 和一个整数 k 。请你从 nums 中满足下述条件的全部子数组中找出最大子数组和：
子数组的长度是 k，且子数组中的所有元素 各不相同 。
返回满足题面要求的最大子数组和。如果不存在子数组满足这些条件，返回 0 。
子数组 是数组中一段连续非空的元素序列。
```

```python
class Solution:
    def maximumSubarraySum(self, nums: List[int], k: int) -&gt; int:
        n, total, window_total, left = len(nums), 0, 0, 0
        dup = set()
        for right in range(n):
            # 存在重复的，从最左边开始收缩窗口，直到不重复
            while nums[right] in dup:
                dup.remove(nums[left])
                window_total -= nums[left]
                left &#43;= 1
            
            dup.add(nums[right])
            window_total &#43;= nums[right]

            if right - left == k-1: # 窗口填满                
                total = max(total, window_total)

                # 窗口滑动
                window_total -= nums[left]
                dup.remove(nums[left])
                left &#43;= 1
        return total
```


---

> 作者: 迷途小狼崽  
> URL: http://localhost:1313/posts/leetcode/%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3/  


# 题目

给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的 连续 子数组，并返回其长度。如果不存在符合条件的子数组，返回 0。

# 解题思路



# 代码

```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        int n = nums.length;
        if (n == 0) {
            return 0;
        }
        int ans = Integer.MAX_VALUE;
        int start = 0;
        int sum = 0;
        for (int end = 0;end < n;end++){
            sum = sum + nums[end];
            while (sum >= s) {
                ans = Math.min(ans, end - start + 1);
                sum = sum - nums[start];
                start = start + 1;
            }
        }
        return ans == Integer.MAX_VALUE ? 0 : ans;

    }
}
```


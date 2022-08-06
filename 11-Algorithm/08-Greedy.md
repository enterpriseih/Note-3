# Greedy

> 适用Greedy的场景：
>
> 简单地说，问题能够分解成子问题来解决，子问题的最优解能够递推到最终问题的最优解。这种最有子问题最优解称为最优子结构。
>
> 贪心算法与动态规划的不同在于它对每个子问题的解决方案都作出选择，不能回退。动态规划则会保存以前的运算结果，并根据以前的运算结果对当前进行选择，有回退功能。

## [122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

```java
//方法一：贪心算法
class Solution {
    public int maxProfit(int[] prices) {
        int ans = 0;
        int n = prices.length;
        for (int i = 1; i < n; ++i) {
            ans += Math.max(0, prices[i] - prices[i - 1]);
        }
        return ans;
    }
}

//方法二：动态规划
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;
        int dp0 = 0, dp1 = -prices[0];
        for (int i = 1; i < n; ++i) {
            int newDp0 = Math.max(dp0, dp1 + prices[i]);
            int newDp1 = Math.max(dp1, dp0 - prices[i]);
            dp0 = newDp0;
            dp1 = newDp1;
        }
        return dp0;
    }
}
```




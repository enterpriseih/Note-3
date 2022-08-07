# Dynamic Programming

> **动态规划:**
>
> 1. 递归+记忆化	=> 递推
> 2. 状态的定义：opt[n] , dp[n] , fib[n]
> 3. 状态转移方程：opt[n] = best_of( opt[n-1] , opt[n-2] , ...)
> 4. 最优子结构
>
> **动态规划 vs 回溯 vs 贪心算法:**
>
> 1. 回溯（递归）：重复计算
> 2. 贪心算法：永远局部最优
> 3. 动态规划：记录局部最优子结构/多种记录值

## [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

```java
class Solution {
    public int climbStairs(int n) {
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;
        for(int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }
}

//👍 
class Solution {
    public int climbStairs(int n) {
        int p = 0, q = 0, r = 1;
        for (int i = 1; i <= n; ++i) {
            p = q; 
            q = r; 
            r = p + q;
        }
        return r;
    }
}
```

## [120. 三角形最小路径和](https://leetcode.cn/problems/triangle/)

**动态规划：**

> 1. 定义： 从下往上递推, 从底走到[i,j]这个点各个路径和的最小值	DP[i,j] = bottom -> [i,j]
> 2. 方程：DP[i,j] = min( DP[i+1,j] , DP[i+1 , j+1] ) + Triangle[i,j]

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        int n = triangle.size();
        int[] f = new int[n];
        f[0] = triangle.get(0).get(0);
        for (int i = 1; i < n; ++i) {
            f[i] = f[i - 1] + triangle.get(i).get(i);
            for (int j = i - 1; j > 0; --j) {
                f[j] = Math.min(f[j - 1], f[j]) + triangle.get(i).get(j);
            }
            f[0] += triangle.get(i).get(0);
        }
        int minTotal = f[0];
        for (int i = 1; i < n; ++i) {
            minTotal = Math.min(minTotal, f[i]);
        }
        return minTotal;
    }
}
```

## [152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)

**动态规划：**

> 1. 定义：DP[i,j] : 表示从0开始到第i个元素最大乘积值，j表示正负，0表示正的最大，1表示负的最大
> 2. 方程：
>    - DP[i,0] = if( a[i] >= 0 ) DP[i-1,0] * a[i]	else	 DP[i-1,1] * a[i]
>    - DP[i,1] = if( a[i] >= 0 ) DP[i-1,1] * a[i]    else     DP[i-1,0] * a[i]

```java
class Solution {
    public int maxProduct(int[] nums) {
        int max = Integer.MIN_VALUE, imax = 1, imin = 1;
        for(int i=0; i<nums.length; i++){
            if(nums[i] < 0){ 
              int tmp = imax;
              imax = imin;
              imin = tmp;
            }
            imax = Math.max(imax*nums[i], nums[i]);
            imin = Math.min(imin*nums[i], nums[i]);
            
            max = Math.max(max, imax);
        }
        return max;
    }
}
```

## [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

> 类似问题：122、123、309、188、714
>
> **动态规划：**
>
> 1. 定义：MP[i，j] 表示到了第 i 天的最大利润, j=0表示手里没有股票，j=1表示手里有股票
> 2. 方程：
>    - MP[i,0] = max( MP[i-1,0] , MP[i-1,1]+a[i] )
>    - MP[i,1] = max( MP[i-1,1] , MP[i-1,0] -a[j] )
>
> **泛化：**
>
> 1. MP[i,k,j]: i表示第 i 天的最大利润， j 表示是否持有股票，k表示已经交易的次数
> 2. MP[i,k,0] = max( MP[i-1,k,0] , MP[i-1,k-1,1] + a[i] )
> 3. MP[i,k,1] = max( MP[i-1,k,1] , MP[i-1,k-1,0] - a[i] )
> 4. Max( MP[n-1,{0...k},0] )

```java
```

## [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)



## [322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

> **动态规划：**类比爬楼梯问题
>
> 1. 定义：DP[i] 到达 i 级台阶最少步数
> 2. 方程：DP[i] = min( DP[i] , DP[i - cons[j] ] + 1 ) 

```java
```



## [72. 编辑距离](https://leetcode.cn/problems/edit-distance/) 👈

> 1. 定义：DP[i,j]表示单词1前i 个字符变到单词2 前j 个字符的最小次数， i表示单词a的第 i 个字符，j 表示单词b的第 j个字符
> 2. 方程：DP[i,j] = if( w1[i] == w2[j] ) DP[i-1,j-1]  else 1+min( DP[i-1,j] , DP[i,j-1] , DP[i-1,j-1] )

```java
```








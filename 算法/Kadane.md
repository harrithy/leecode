# Kadane算法

## 算法简介

Kadane算法是一种用于解决**最大子数组和问题**的经典动态规划算法喵～它由美国卡内基梅隆大学的Joseph Born Kadane教授在1984年提出，以其简洁优雅和高效著称呢！

这个算法的核心思想其实很简单：**对于数组中的每个位置，我们只需要做一个决策——是把当前元素加入前面的子数组，还是从当前元素重新开始一个新的子数组呀？**

## 问题定义

给定一个整数数组 `nums`，找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```txt
输入：nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
输出：6
解释：连续子数组 [4, -1, 2, 1] 的和最大，为 6
```

## 算法思想

### 核心洞察

Kadane算法的精髓在于一个简单但深刻的观察喵：

> **如果前面累积的和是负数，那它对后面只会是拖累，不如丢掉重新开始！**

换句话说呢：
- 如果 `前面的累积和 > 0`：前面是**正贡献**，保留它，继续累加
- 如果 `前面的累积和 ≤ 0`：前面是**负贡献**或零贡献，丢弃它，从当前位置重新开始

### 状态定义

设 `dp[i]` 表示**以 `nums[i]` 结尾**的最大子数组和。

⚠️ 注意这里的定义是"以 `nums[i]` **结尾**"，而不是"前 i 个元素中的最大子数组和"喵！这个定义方式保证了子数组的**连续性**，是理解Kadane算法的关键呢～

### 状态转移方程

```
dp[i] = max(dp[i-1] + nums[i], nums[i])
```

翻译成人话就是：
- **选项A**：`dp[i-1] + nums[i]` — 把 `nums[i]` 接到前面的子数组后面
- **选项B**：`nums[i]` — 从 `nums[i]` 重新开始一个新子数组

哪个大选哪个，就这么简单喵～

### 图解过程

以 `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]` 为例：

```
索引 i:    0    1    2    3    4    5    6    7    8
nums[i]:  -2    1   -3    4   -1    2    1   -5    4
dp[i]:    -2    1   -2    4    3    5    6    1    5
maxSum:   -2    1    1    4    4    5    6    6    6

详细过程：
i=0: dp[0] = -2（只有一个元素，就是它本身）, maxSum = -2
i=1: dp[1] = max(-2+1, 1) = max(-1, 1) = 1（重新开始！）, maxSum = 1
i=2: dp[2] = max(1+(-3), -3) = max(-2, -3) = -2（继续累加）, maxSum = 1
i=3: dp[3] = max(-2+4, 4) = max(2, 4) = 4（重新开始！）, maxSum = 4
i=4: dp[4] = max(4+(-1), -1) = max(3, -1) = 3（继续累加）, maxSum = 4
i=5: dp[5] = max(3+2, 2) = max(5, 2) = 5（继续累加）, maxSum = 5
i=6: dp[6] = max(5+1, 1) = max(6, 1) = 6（继续累加）, maxSum = 6 ✨
i=7: dp[7] = max(6+(-5), -5) = max(1, -5) = 1（继续累加）, maxSum = 6
i=8: dp[8] = max(1+4, 4) = max(5, 4) = 5（继续累加）, maxSum = 6
```

最终答案是 6，对应子数组 `[4, -1, 2, 1]` 喵～

## 代码实现

### 基础版本（完整dp数组）

```javascript
/**
 * Kadane算法 - 基础版本
 * @param {number[]} nums
 * @return {number}
 */
function kadaneBasic(nums) {
    const n = nums.length;
    // dp[i] 表示以 nums[i] 结尾的最大子数组和
    const dp = new Array(n);
    
    // 初始化：第一个元素结尾的最大子数组和就是它本身
    dp[0] = nums[0];
    let maxSum = dp[0];
    
    for (let i = 1; i < n; i++) {
        // 核心决策：加入前面 or 重新开始
        dp[i] = Math.max(dp[i - 1] + nums[i], nums[i]);
        // 更新全局最大值
        maxSum = Math.max(maxSum, dp[i]);
    }
    
    return maxSum;
}

// 时间复杂度：O(n)
// 空间复杂度：O(n)
```

### 优化版本（空间优化）

由于 `dp[i]` 只依赖于 `dp[i-1]`，我们可以用一个变量代替整个数组喵～

```javascript
/**
 * Kadane算法 - 空间优化版本
 * @param {number[]} nums
 * @return {number}
 */
function kadaneOptimized(nums) {
    // pre 表示以前一个位置结尾的最大子数组和
    let pre = 0;
    // maxSum 记录全局最大值
    let maxSum = nums[0];
    
    for (let i = 0; i < nums.length; i++) {
        // 核心决策：加入前面 or 重新开始
        pre = Math.max(pre + nums[i], nums[i]);
        // 更新全局最大值
        maxSum = Math.max(maxSum, pre);
    }
    
    return maxSum;
}

// 时间复杂度：O(n)
// 空间复杂度：O(1) ✨
```

### 进阶版本（返回子数组本身）

如果不仅要求最大和，还要返回具体是哪个子数组呢？

```javascript
/**
 * Kadane算法 - 返回最大子数组本身
 * @param {number[]} nums
 * @return {{maxSum: number, subarray: number[]}}
 */
function kadaneWithSubarray(nums) {
    let pre = 0;
    let maxSum = nums[0];
    
    // 记录当前子数组的起始位置
    let tempStart = 0;
    // 记录最大子数组的起始和结束位置
    let start = 0;
    let end = 0;
    
    for (let i = 0; i < nums.length; i++) {
        // 如果重新开始更好，更新临时起始位置
        if (pre + nums[i] < nums[i]) {
            pre = nums[i];
            tempStart = i;
        } else {
            pre = pre + nums[i];
        }
        
        // 更新全局最大值和对应的位置
        if (pre > maxSum) {
            maxSum = pre;
            start = tempStart;
            end = i;
        }
    }
    
    return {
        maxSum: maxSum,
        subarray: nums.slice(start, end + 1)
    };
}

// 示例
// kadaneWithSubarray([-2,1,-3,4,-1,2,1,-5,4])
// 返回：{ maxSum: 6, subarray: [4, -1, 2, 1] }
```

## 复杂度分析

| 版本 | 时间复杂度 | 空间复杂度 |
|------|-----------|-----------|
| 基础版本 | O(n) | O(n) |
| 优化版本 | O(n) | O(1) |
| 进阶版本 | O(n) | O(1) |

Kadane算法只需要遍历数组一次，是解决最大子数组和问题的最优解法喵～

## 算法变体与应用

### 变体1：最大子数组积

类似的思想可以用于求最大子数组积，但需要同时维护最大值和最小值（因为负负得正呀）：

```javascript
function maxProduct(nums) {
    let maxProd = nums[0];
    let minProd = nums[0];
    let result = nums[0];
    
    for (let i = 1; i < nums.length; i++) {
        // 如果当前是负数，最大最小会交换
        if (nums[i] < 0) {
            [maxProd, minProd] = [minProd, maxProd];
        }
        
        maxProd = Math.max(nums[i], maxProd * nums[i]);
        minProd = Math.min(nums[i], minProd * nums[i]);
        
        result = Math.max(result, maxProd);
    }
    
    return result;
}
```

### 变体2：环形数组的最大子数组和

对于环形数组，最大子数组要么是普通的连续子数组，要么是"首尾相连"的部分。后者等价于 `总和 - 最小子数组和`：

```javascript
function maxSubarraySumCircular(nums) {
    let maxSum = nums[0], minSum = nums[0];
    let curMax = 0, curMin = 0;
    let total = 0;
    
    for (const num of nums) {
        curMax = Math.max(curMax + num, num);
        maxSum = Math.max(maxSum, curMax);
        
        curMin = Math.min(curMin + num, num);
        minSum = Math.min(minSum, curMin);
        
        total += num;
    }
    
    // 如果所有元素都是负数，返回最大元素
    // 否则返回 max(普通最大, 环形最大)
    return maxSum > 0 ? Math.max(maxSum, total - minSum) : maxSum;
}
```

### 相关LeetCode题目

| 题号 | 题目 | 难度 |
|------|------|------|
| 53 | 最大子数组和 | 中等 |
| 152 | 乘积最大子数组 | 中等 |
| 918 | 环形子数组的最大和 | 中等 |
| 1186 | 删除一次得到子数组最大和 | 中等 |

## 为什么Kadane算法是正确的？

这里稍微严谨一点解释一下喵～（小傲娇：虽然直觉上很显然，但证明一下也不错呢）

**证明思路**：

1. 最大子数组一定以某个位置 `k` 结尾
2. `dp[k]` 正好就是以位置 `k` 结尾的最大子数组和
3. 我们遍历了所有位置，取了所有 `dp[i]` 的最大值
4. 所以一定能找到全局最大的子数组和 QAQ

**状态转移的正确性**：

对于以 `nums[i]` 结尾的子数组，只有两种情况：
- 长度为1：就是 `nums[i]` 本身
- 长度大于1：一定是某个以 `nums[i-1]` 结尾的子数组 + `nums[i]`

而以 `nums[i-1]` 结尾的最大子数组和就是 `dp[i-1]`，所以：

```
dp[i] = max(nums[i], dp[i-1] + nums[i])
```

这个转移方程是完备的喵～

## 总结

Kadane算法的精髓可以用一句话概括：

> **走一步看一步，前面是累赘就丢掉，轻装上阵才能走得更远喵～**

核心要点：
1. **状态定义**：`dp[i]` 是以 `nums[i]` **结尾**的最大子数组和
2. **决策思想**：前面是正贡献就保留，是负贡献就丢弃
3. **空间优化**：状态只依赖前一个，可以用单变量代替数组
4. **时间复杂度**：O(n)，一次遍历搞定

这个算法虽然简单，但思想非常优雅呢！理解了它，很多类似的DP问题都能迎刃而解呀～ ✨

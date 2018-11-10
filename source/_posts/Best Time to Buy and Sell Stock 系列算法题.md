title: Best Time to Buy and Sell Stock 系列算法题
date: 2017-04-17 23:04:03
tags: 算法, LeetCode, swift, 动态规划
categories: algorithm
---
这是最近在学算法中完成的第一个系列题，系列题的特点是后一道是前一道的升级版，后一道比前一道难，基本是一道比一道有意思，所以写一下总结是必须的呀。那为啥要重新学算法，大概是因为之前学得水吧。其实最近的计划是保持进阶学习的同时，不断地巩固基础，不断地进行精深练习。然后刚好遇到一个大神主播最近在讲算法，那就跟着学吧。

<!--more-->

好了，今天的主角是LeetCode上的这几道题：
121	Best Time to Buy and Sell Stock 
122	Best Time to Buy and Sell Stock II
123	Best Time to Buy and Sell Stock III
188	Best Time to Buy and Sell Stock IV
309	Best Time to Buy and Sell Stock with Cooldown   

其实主角是应该是动态规划，这类最佳xx的题目，一看就先想到动态规划。每一道题大概我都会写下我当时的思路，和怎么不通过，怎么改进，怎么优化。

#### 121 Best Time to Buy and Sell Stock 
[原题地址](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/#/description)。题目的意思大概是，给你一个股票的每天价格数组prices，你在只能进行一次操作（即一次买入和一次卖出）的限制下，求最大的收益。比如每天的价格为[7, 1, 5, 3, 6, 4]，那么最大收益就是1的时候买入，6的时候卖出，这样就能得到最大收益5。

第一步，明确下边界，当输入为空数组时，那最大收益就是0。而每天的价格，肯定是正数。

第二步，考虑不同的解决方案，这道题，用动态规划肯定是可以解的，然后其他方案，我也想不出来。
动态规划，先定义个dp[i]来表示第i天的最大收益，
dp[i] = prices[i] - prices[j]第j天买入的价格，因为交易只能有一次，所以，当买入卖出完成后，就能缓存当前结果了。
用swift来写的话，大概的递推过程如下：
```js
var maxIncome = 0
for i in 0 ..< prices.count {
    for j in 0 ..< i {
        if prices[i] - prices[j] > maxIncome {
            maxIncome = prices[i] - prices[j]
        }
    }
}
return maxIncome
```
这样就能算出最大的收益了，不过如果把这段代码方式LeetCode上跑，结果是超时的。为啥呢？
额，在解释前，我说个LeetCode傻逼的地方，因为swift比较新吧，然后版本更迭又挺频繁的，所以swift的代码，LeetCode上很难保证其运行时间，比如说在同样时间复杂度的解题方案下，用JavaScript写的能过，用swift的不能过，额，真是坑爹。我被这个坑了几个小时，害我不断地优化代码，所以，虽然我是用swift来解题，但是有时候，我是用js复写了一遍swift的代码然后来提交的。
好了，上面这个解法为啥超时呢。。。根据经验，应该是重复计算的问题，但其实这里并没有重复计算，而是计算了很多本不需要计算的结果。第二个循环其实很多计算是不必要的，因为，我们要得出当前差值最大，那肯定是减去在i之前最小的值，而不需要去减i之前的每个数
所以，优化后，代码如下：
```js
var maxIncome = 0
var minValue = Int.max
for i in 0 ..< prices.count {
    if prices[i] < minValue {
        minValue = prices[i]
    }
    if i > 0 {
        if prices[i] - minValue > maxIncome {
            maxIncome = prices[i] - minValue
        }
    }
}
return maxIncome
```
上面的两个方案，从复杂度来看，一个是O(n^2)，一个是O(n)，然后提交，accepted!!! 

第三步，优化代码
其实这一步在第二步已经进行过一次了，即复杂度的优化，接下来就是一些小的优化，和代码风格的优化。
1、minValue的初始值可以为第一天的价格
2、在递推过程中，可以从第2天的开始，即i=1的时候。
```js
var maxIncome = 0
var minValue = prices[i]
for i in 1 ..< prices.count {
    if prices[i] < minValue {
        minValue = prices[i]
    }
    if prices[i] - minValue > maxIncome {
        maxIncome = prices[i] - minValue
    }
}
return maxIncome 
```
上面的代码可以用三目运算符继续缩减行数的，不过，我觉得这样已经差不多了。ps，上面3段代码都是重新手打的，不保证通过啊。


#### 122 Best Time to Buy and Sell Stock II
[原题地址](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/#/description)。这道题是在上题的基础上，然后不限制交易次数，但是每次交易不能同时进行，即买入后要卖出才能进行下一次买入的操作。
第一道题已经把基本的、通用的结果步骤写出来了，我解题的时候大概也是遵循这样的步骤来解，所以第二题就不详细啰嗦的写了。
动态规划的第一步是给出状态的定义，在这道题中
我们这样来定义最大收益dp：
```js
dp[i][0]，dp[i][1]代表 the i day, have or not stock，即第i天是否拥有股票哟，0代表没有，1代表当天持有股票。
```
第二步是给出状态递推方程
```js
dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i])  
//即第i天没股票的最大收益，dp[i-1][0]代表前一天没有股票，dp[i-1][1]+prices[i]前一天有股票并卖出，而第i天的最大收益会是这两者之间的最大值。
dp[i][1] = max(dp[i - 1][0] - prices[i], dp[i - 1][1])  
//即第i天持有股票时的最大收益，dp[i-1][0]-prices[i]代表前一天没有股票并买入，dp[i-1][1]表示前一天持有股票的最大收益
```
有了上面的递推方程，即dp[prices.count-1][0]，最后一天没有股票，就是当前的最大收益了。

其实这里可能大家会有疑惑，为啥dp要是二维数组，为什么一个为天数，一个为是否持股。额，其实，动态规划就是把每个因素转化为一个维度，这是标准的动态规划递推方程，每当有不同的决定因素时，往往可以通过增加维度来解题。但是增加维度的方式有多种，一位数组变成二维数组是一个方法，而增加不同的变量也是可以达到增加维度的效果的，如这里，你可以定义，s0[i],s1[i]分别为第i天没持股，和持股的最大收益。这样就不需要二维数组啦，但是方法的本质是一样。

代码如下（swift）：
```js
func maxProfit(_ prices: [Int]) -> Int {
    if prices.count <= 1 { return 0 }
    var dp: [[Int]] = Array(repeating: [0, 0], count: prices.count)
    dp[0][0] = 0
    dp[0][1] = -prices[0]
    for i in 1 ..< prices.count  {
        dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i])
        dp[i][1] = max(dp[i - 1][0] - prices[i], dp[i - 1][1])
    }
    return dp[prices.count - 1][0]
}
```

这道题还有个神奇的解法，不过跟动态规划没啥关系，我是看着别人代码写出来的😂，自行理解吧：
```js
func maxProfit(_ prices: [Int]) -> Int{
    var max = 0
    for i in 1 ..< prices.count {
        if prices[i] > prices[i - 1] {
            max += prices[i] - prices[i - 1]
        }
    }
    return max
}
```

#### Best Time to Buy and Sell Stock III 和 IV
[原题III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/#/description)
[原题IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/#/description)

第三题是限制次数两次，第四题是限制交易次数k次，在解第三题的时候，可以用一些方式，达到不增加维度的方式来解题，具体参照LeetCode讨论区里面的解法吧。而第四题是比第三题更加具有普遍性，上面说过，如果有新的决定因素，我们可以将它作为一个新的维度。即：
//dp[i][j][0]  dp[i][j][1]   the i day, have j transactions,  have or not stock
//dp[i][j][0] = max(dp[i - 1][j][0], dp[i - 1][j - 1][1] + prices[i])
//dp[i][j][1] = max(dp[i - 1][j][0] - prices[i], dp[i - 1][j][1])
第一行是状态定义，跟之前的定义类似，但是增加了一个维度，即第i天已经有j次交易是否持股的最大收益。
第二、第三行是状态转移方程，首先，买入股票并没有完成一次操作，而是在卖出之后，才会增加交易次数。所以能得出上面的两个方程。
代码如下：
```js
func maxProfit(_ k: Int, _ prices: [Int]) -> Int {
    if prices.count <= 1 { return 0 }

    var dp: [[[Int]]] = Array(repeating: Array(repeating: [Int.min, Int.min], count: k + 1), count: prices.count)
    dp[0][0][0] = 0
    dp[0][0][1] = -prices[0]
    
    var maxIncome = 0
    for i in 0 ..< prices.count  {
        for j in 0 ... k {
            if i > 0 {
                if j > 0 {
                    dp[i][j][0] = max(dp[i - 1][j][0], dp[i - 1][j - 1][1] + prices[i])
                } else {
                    dp[i][j][0] = 0
                }
                dp[i][j][1] = max(dp[i - 1][j][0] - prices[i], dp[i - 1][j][1])

                maxIncome = max(maxIncome, dp[i][j][0])
            }
        }
    }
    return maxIncome
}
```
maxIncome用来缓存j次交易后的最大收益，最后就是最大收益。上面的代码其实是不能通过, 但是这并不是重点。上面的代码大概就代表了整个解题的思路和递推过程。

假如理解了上面的代码，再来理解下面完整的代码，其实整体没啥变化，加了一些优化而已：
```js
func maxProfit(_ k: Int, _ prices: [Int]) -> Int {
    if prices.count <= 1 { return 0 }
    
    if k > prices.count / 2 {
        return maxIncomeK(prices)
    }
    
    var dp: [[[Int]]] = Array(repeating: Array(repeating: [Int.min, Int.min], count: k + 1), count: 2)
    dp[0][0][0] = 0
    dp[0][0][1] = -prices[0]
    
    var maxIncome = 0
    for i in 1 ..< prices.count  {
        let x = i % 2
        let y = (i - 1) % 2
        for j in 0 ... k {
            if j > 0 {
                dp[x][j][0] = max(dp[y][j][0], dp[y][j - 1][1] + prices[i])
            } else {
                dp[x][j][0] = 0
            }
            dp[x][j][1] = max(dp[y][j][0], dp[y][j][1] + prices[i]) - prices[i] //避免Int.min减去某个数，导致的溢出崩溃。
            maxIncome = max(maxIncome, dp[x][j][0])
        }
    }
    return maxIncome
}

func maxIncomeK(_ prices: [Int]) -> Int{
    var max = 0
    for i in 1 ..< prices.count {
        if prices[i] > prices[i - 1] {
            max += prices[i] - prices[i - 1]
        }
    }
    return max
}
```
上面的代码变量名、方法名有些不是起得很合理，词穷啊，想名字好难。上面的代码对比之前的方案，差别在于：
1、当k > prices.count / 2，这个时候就变回第二题了，这里的maxIncomK是第二题中那个神奇的解法。
其实完成上面这步处理，这道题就算完了，但是还有优化的地方，这里不能理解就略过吧，不过连我都能理解，应该没啥难度
2、状态压缩：var dp在定义的时候，第一维度只用来2个空间大小，而let x = i % 2和let y = (i - 1) % 2则是配套使用，额，其实本质是因为maxIncome用来缓存了当前的最大收益，所以这里可以不用那么多空间来存储，只需要用到这次跟上一次，这两个存储空间就够用了，这里属于空间复杂度的优化。
3、在递推过程里面，dp[x][j][1]的方程是不是有点奇怪，其实这是为了解决swift中Int.min减去某个数，导致的溢出崩溃。因为我们初始化的时候把每个值都初始化为Int.min。
好像没其他了，这道题就这样吧。


#### 309 Best Time to Buy and Sell Stock with Cooldown
这里每次交易完成后，必须冷静下，即至少暂停一天的操作，不能买不能卖，冷静一天都才有所决策。
这道题其实难度比3、4小，如果3、4都理解了，那么这道题就不难理解了。
下面给出我的解题方案（js）：
```js
/**
 * @param {number[]} prices
 * @return {number}
 */
var maxProfit = function(prices) {
    if (prices.length <= 1) return 0 
    var dp = []
    for (let i = 0; i < prices.length; i++) {
        dp[i] = []
    }
    
    dp[0][0] = 0
    dp[0][1] = -prices[0]
    dp[0][2] = Number.MIN_VALUE
    for (let i = 1; i < prices.length; i++)  {
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][2])
        dp[i][1] = Math.max(dp[i - 1][0] - prices[i], dp[i - 1][1])
        dp[i][2] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i])
    }
    return Math.max(dp[prices.length - 1][0], dp[prices.length - 1][2])
}
```
为啥我用了js来写，因为就是最前面说的LeetCode上swift抽风的问题。
dp[i][0] 表示第i天没有持股的最大收益
dp[i][1] 表示第i天有持股的最大收益
dp[i][2] 表示第i天没有持股，并且刚刚卖出了股票，如果是这个状态，那接下来就要冷静啦。即不能马上持股。


#### 写在最后
这几道题目在写完之后确实收获不小，建议大家都亲手写一遍吖。

有人说算法没什么鸟蛋用，额，其实，写算法跟写业务差不多，好像这句我说过啊。
按我目前的理解，算法是进行精深练习的一个比较方便快捷的途径，不过也是比较原始的。但这能不断强化一些基本的代码能力，思考和抽象的能力吧。而且算法作为计算机的基础课程，对以后进阶有很大帮助吖。
上面这段我乱说的，大家自行理解吧。

还有一个，说到动态规划，我之前二话不说就是一个递归。但其实动态规划更应该用递推而不是递归。
所以以后看到动态规划的题，还是用递推吧。
Prefix Skill:

前缀技巧切入点是想要快速获取某个区间的状态，发散一下，某个区间可以是数组的 [j, i)区间， 可以是树遍历下的某个子遍历区间等

状态可以是 【和状态】， 也可以是【区间中某个值的个数状态】等

当解题思路递进道拥有👆的想法时候，就直接套模版吧, 因为使用前缀技巧，更多是细节的处理和场景的处理，比如区间范围的划分，
通过什么数据结构保存状态，以便于快速查找，选用的数据结构存什么，可以满足我们需要快速查找的条件，才是比较难的
部分。

而前缀的模板是什么呢 ？ (这里插一句：模板现在在我这里的定义可以理解为数据公式，上升到道义，万物皆数学)

```go
// 大多数情况下可以是这个抽象
 Status[j,i) = Status[0, i) - Status[0, j)
设 ： prefixStatus[x] = Status[0,x)
==> Status[j,i) = prefixStatus[i] - prefixStatus[j]
//解释一下这里用左开右闭的原因是，[0,0) 可以处理边界值的情况，让获取前缀状态可以从0开始，[0,0) 代表没有值
(i-j) 代表 [j, i) 的状态
```

为了体现前缀思路的层次递进，从最纯粹的开始，慢慢到变式
```go
//724. 寻找数组的中心下标
//给你一个整数数组 nums ，请计算数组的 中心下标 。
//
//数组 中心下标 是数组的一个下标，其左侧所有元素相加的和等于右侧所有元素相加的和。
//
//如果中心下标位于数组最左端，那么左侧数之和视为 0 ，因为在下标的左侧不存在元素。这一点对于中心下标位于数组最右端同样适用。
//
//如果数组有多个中心下标，应该返回 最靠近左边 的那一个。如果数组不存在中心下标，返回 -1 。


抽象的状态是：f(idx) = {sum[0, idx) == sum(idx, len(nums))}
公式：Status[idx] = sum[0, idx) == sum(idx, len(nums)) 为true
设: prefixSum[x] = sum[0, x)
则： sum[0, idx) == sum(idx, len(nums))
	==> prefixSum[idx] == prefixSum[len(nums)] - prefixSum[idx] - nums[idx]


func f(nums []int) int {
	prefixSum := make([]int, len(nums)+1) // +1 是为了 用prefixSum[0] 保存 原数组 [0,0) 的状态, 原数组中[0, 0) 这个时候其实是没状态的
	for idx, it := range nums {
		prefixSum[idx+1] = prefixSum[idx] + it
    }
	
	for idx := 0; idx < len(nums); idx++ {
		//sum[0, idx) == sum(idx, len(nums))
	    if prefixSum[idx] == prefixSum[len(prefixSum)-1] - prefixSum[idx] - nums[idx] {
			return idx
		} 
    }
	return -1
}
```

不知道看完👆的简单题，是否对数据公示的魔力有所震撼呢？反正我写完我是服了哈哈哈, 让我们keep going on
```go
//238. 除自身以外数组的乘积
//给你一个整数数组 nums，返回 数组 answer ，其中 answer[i] 等于 nums 中除 nums[i] 之外其余各元素的乘积 。
//
//题目数据 保证 数组 nums之中任意元素的全部前缀元素和后缀的乘积都在  32 位 整数范围内。
//
//请不要使用除法，且在 O(n) 时间复杂度内完成此题。

抽象的状态是：f(idx) = {multiply[0, idx) * multiply(idx, len(nums))}
公式：Status[idx] = multiply[0, idx) * multiply(idx, len(nums)) 
设: prefixProduct[x] = multiply[0, x)
则： multiply[0, idx) * multiply(idx, len(nums))

到这里简直和上面的一模一样，但是可能这题困难一丢丢的地方就在于multiply(idx, len(nums)) 如何 表示吧
'.'  not use divided 
.'.  multiply(idx, len(nums)) != prefixProduct[len(nums)]/prefixProduct[idx]
     那就只能打直球了，直接 求 multiply(idx, len(nums)) ，但是每次循环求 (idx, len(nums))的乘积是不是也和prefixProduct 一样了
.'.  设 suffixProduct[idx] = multiply[idx, len(nums))， 还是为了可以让suffixProduct处理边界情况

multiply[0, idx) * multiply(idx, len(nums)) 
==> prefixProduct[idx] * suffixProduct[idx+1] 

func f(nums []int) []int {
    prefixProduct := make([]int, len(nums)+1) // +1 是为了 用prefixSum[0] 保存 原数组 [0,0) 的状态, 原数组中[0, 0) 这个时候其实是没状态的
    suffixProduct := make([]int, len(nums)+1) 
	prefixProduct[0] = 1
	suffixProduct[len(nums)] = 1
    for idx, it := range nums {
        prefixSum[idx+1] = prefixSum[idx] * it
		suffixProduct[len(nums)-idx-1] = suffixProduct[len(nums)-idx] * it
    }
    res := make([]int, len(nums))
    for idx := 0; idx < len(nums); idx++ {
		res[idx] = prefixSum[idx] * suffixProduct[idx+1]
    }
    return res
}
```

以上可能都还是用一个prefix array 可以搞定的，现在我们进阶一些需求

```go
//525. 连续数组
//给定一个二进制数组 nums , 找到含有相同数量的 0 和 1 的最长连续子数组，并返回该子数组的长度。

Status[i, j) = cnt(0) == cnt(1) => cnt(0) - cnt(1) == 0
难点在于怎么抽象这个区间的状态到前缀的技巧上， 其实关键在于这是一个 cnt(0) == cnt(1) 等式，与👆的等式不同的是这个等式两边不太好一次性求出，
前面的等式两边都直接或者间接和prefix 挂钩，这里就要灵活转一下，将原数组的0变成-1, 然后这个等式就可以比较巧妙的转成
Status[i,j) = sum[i, j) == 0 
设：prefixSum[x] = sum[0, x)
则： Status[i,j) = sum[i, j) == 0 ==> prefixSum[j] - prefixSum[i] == 0

code 略，so easy

//523. 连续的子数组和
//给你一个整数数组 nums 和一个整数 k ，编写一个函数来判断该数组是否含有同时满足下述条件的连续子数组：
//
//子数组大小 至少为 2 ，且
//子数组元素总和为 k 的倍数。
//如果存在，返回 true ；否则，返回 false 。
//
//如果存在一个整数 n ，令整数 x 符合 x = n * k ，则称 x 是 k 的一个倍数。0 始终视为 k 的一个倍数。
//1 <= nums.length <= 10^5
//0 <= nums[i] <= 10^9
//0 <= sum(nums[i]) <= 2^31 - 1
//1 <= k <= 2^31 - 1


Status[j, i) = sum[j, i) % k == 0 && i-j>=2
设: prefixSum[x] = sum[0, x) 
则：sum[j, i) % k == 0 && i-j>=2
==> (prefixSum[0, i) - prefixSum[0, j)) % k == 0 && i-j>=2
==> prefixSum[0, i) % k == prefixSum[0, j) % k && i-j>=2
这里困难可能就是在于 如何 求的j 存在与否了吧，与👆不同的是，上面只需要求的 i 下满足某个条件就行，这里
是需要求得 [j, i) 满足的条件, 并且i-j>=2 也就是说，在能很快得到 prefixSum[0, i) 的情况咋很快得到 prefixSum[0, j) 结合
满足上面的式子, 当然hashMap 就很好想到了

那hashMap 存什么，做什么，我们需要快速找到是j ，那 value 一定是true or false, key 其实就是 prefixSum[0, j) % k,
最后map 判断就行了, 这里再讲一个小技巧，查map 的过程是向前看的过程，可以边求prefix， 边查map，这个技巧如果你要问我
怎么想到的，我只能说，优化的思想一定要有，其次就是熟能生巧，别的没了

func f(nums []int, k int)  bool {
    prefixSum, prefixSumMap := make([]int, len(nums)+1), make(map[int]struct{})// i-2 >= j
    //  0 <= sum(nums[i]) <= 2^31 - 1 可以用int 表示 hashMap 中的key, 因为value 只需要T/F
    for idx, it := range nums {
      prefixSum[idx+1] = prefixSum[idx]+ it
    }
    for idx := 1; idx < len(nums); idx ++ {
        prefixSumMap[prefixSum[idx-1]%k] = struct{}{} // i - j 之间需要有一定的区间差
        //  prefixSum[0, j) % k 这个key 是否存在 所以可以在这个时候判断
        if _, exist := prefixSumMap[prefixSum[idx+1]%k]; exist {
        return true
        }    
	}
    return false
}

//560. 和为 K 的子数组
//给你一个整数数组 nums 和一个整数 k ，请你统计并返回 该数组中和为 k 的连续子数组的个数 。
//1 <= nums.length <= 2 * 10^4
//-1000 <= nums[i] <= 1000
//-10^7 <= k <= 10^7

乍一看还是一个求区间和的算法，但是注意这里要求的是返回不在是某个数组中的值或者bool，而是一个统计值，统计值就代表数组中可能多个区间存在同一个状态
，这就比较有意思，虽然整体思路还是求和，还是算出满足下面的等式的状态

prefixSum[i] - prefix[j] == k

但是prefixSum[j] = sum[0, j) 这个状态会有多个区间，但是还好是统计，所以 hashMap 中value 可以存的是 sum[0, j) 状态的个数， 满足的加上
这个个数即可

func f(nums []int, k int) int {
	prefixSum, prefixSumMap := 0, make(map[int]int, 0)
	res := 0
	prefixSumMap[0] = 1 // 注意prefix[0] ==> 代表有一个状态 [0, 0) ==> 对应的map就是1
	for i := 0; i < len(nums); i ++ {
		prefixSum += nums[i]
		res += prefixSumMap[prefixSum - k]
		prefixSumMap[prefixSum] ++ 
    }
	return res
}

//974. 和可被 K 整除的子数组
//给定一个整数数组 nums 和一个整数 k ，返回其中元素之和可被 k 整除的（连续、非空） 子数组 的数目。
//
//子数组 是数组的 连续 部分。
// 1 <= nums.length <= 3 * 104
//-104 <= nums[i] <= 104
//2 <= k <= 104

再乍一看，发现不就是👆两道题的结合题嘛，满足区间和是k 的倍数，并且统计存在的情况，为什么这个可以有统计值出现呢？其实就是因为原数组可正可负，所以满足
某个状态的区间可以有多个
与上上题不同在于i-j>=1 即可，一个值也是满足非空，
与上题不同的是 需要满足的条件是 % ,不是简单的和，并且在取余的情况下, 需要判断取余之后为负数一定要+k , 满足他hit在正数上
负数的判断需要特别小心，其他的很相似,那就直接上code

func f(nums []int, k int )  int {
    prefixSum, prefixSumMap := 0, make(map[int]int, 0)
	prefixSumMap[0] = 1
	res := 0
	for i := 0; i < len(nums); i ++ {
		prefixSum += nums[i]
		if prefixSum % k < 0 {
			res += prefixSumMap[prefixSum % k + k]
            prefixSumMap[prefixSum % k + k] ++
         }else {
            res += prefixSumMap[prefixSum % k]
			prefixSumMap[prefixSum % k] ++
         }
    }
}
```

在一些题中，如何抽丝剥茧到我想要求区间状态 ？prefix 技巧的抽象思路养成。
```go
//1124. 表现良好的最长时间段
//给你一份工作时间表 hours，上面记录着某一位员工每天的工作小时数。
//
//我们认为当员工一天中的工作小时数大于 8 小时的时候，那么这一天就是「劳累的一天」。
//
//所谓「表现良好的时间段」，意味在这段时间内，「劳累的天数」是严格 大于「不劳累的天数」。
//
//请你返回「表现良好时间段」的最大长度。
//输入：hours = [9,9,6,0,6,6,9]
//输出：3
//解释：最长的表现良好时间段是 [9,9,6]。

其实这题也蛮简单的，切入点是要求的区间状态「劳累的天数」是严格 大于「不劳累的天数」如何用数学公式表达，类似与有点像求区间中某个数据存在的个数这类
的题。
判定劳累的一天是1， 不劳累的一天是-1 ，然后符合条件的区间状态就是这个区间和大于0的最大长度
sum[j, i) > 0 ==> prefixSum[i] - prefixSum[j] > 0 ==> res = max(res, (i-j))
但是有一些细节需要考虑

func longestWPI(hours []int) int {
    for i , _ := range hours {
        if hours[i] <= 8 {
            hours[i] = -1
        }else {
            hours[i] = 1
        }
	}
	res := 0
	prefixSum := 0
	prefixSumMap := make(map[int]int, 0)
	prefixSumMap[0] = 0
	for i := 1; i <= len(hours); i ++ {
        prefixSum += hours[i-1]
		// 这里就需要分正负数讨论， prefixSum 为负数，需要找到一个prefixSum[i] - prefixSum[j] > 0 
		//   ==> prefixSum[j] = prefixSum[i]-1  是否存在？ prefixSum[i] > prefixSum[i]-1 恒成立, 在这里可以这么干, 但是最好的方式
		//  还是通过单调栈求 满足 prefixSum[j] < prefixSum[i] 的 j 
        if prefixSum > 0 { // 
            res = max(res, i)
        }else if d , exist := prefixSumMap[prefixSum-1]; exist {
            res = max(res, (i-d))
        }
        if _, exist := prefixSumMap[prefixSum] ; !exist {
            prefixSumMap[prefixSum] = i
        }
    }
    return res
}

func max(a, b int) int {
if a > b {
return a
}
return b
}


//437. 路径总和 III
//给定一个二叉树的根节点 root ，和一个整数 targetSum ，求该二叉树里节点值之和等于 targetSum 的 路径 的数目。
//
//路径 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。
相当于求 树 的某个 区间的和， 采用遍历解决即可

// 求数组中的某个区间和满足在[x, y] 之间
// 这题有意思的地方在于 状态的满足式子 有两个 sum[j, i) >= x && sum[j, i) <= y 跟上面的只有一个满足条件稍显不同
// 所以这道题的关键是 prefixSum[i] 求的后 ，如何求 j 是满足
//   prefixSum[0, i) - prefixSum[0, j) >= x && prefixSum[0, i) - prefixSum[0, j) <= y ?
//   

```



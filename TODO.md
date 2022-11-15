Nov. 12-14th . 2022
1. [] DataStructure(Array):Prefix skill

   leetcode:
 
    ~~连续的子数组和(523)~~
   
    ~~连续数组(525)~~ 【寻找长度至少为 2 且和为 k 的倍数的子数组】(prefix + map， map 需要想一想存入什么key)

    ~~表现良好的最长时间段(1124)~~ （prefix + map / prefix + stack）

    ~~和为 K 的子数组个数 🟠(560)~~ (prefix + map , map 需要想一下存入什么value)

    ~~和可被 K 整除的子数组的个数， 数组中有负数(974) (prefix + map， 需要考虑prefixSum 是负数的情况)~~

     ~~统计「优美子数组」(满足k个奇数的子数组) 的个数 (1248) (关键一定是需要知道 prefix + map, map 的value 存入什么)~~
 
    ~~路径总和 III (437) 在树上找到满足target的路径(不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点))的和 (结合树遍历做prefix)~~

     ~~乘积小于 K 的子数组(713) 的个数 (prefix + binarySearch , 需要注意的是乘积防止溢出，通过log 对数，将乘积便成log对数的和，最后通过logPrefixSum 数组中寻找第一个下小于 logK的值)~~

2. [] Algorithm:
3. [] Database:
   
      make starrocks run locally

      SQL (1. 三值运算；2. union / union all / or 3. not in / NULL 4. join)
Nov. 12-14th . 2022
1. [x] DataStructure(Array):Prefix skill

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

Nov. 15th . 2022
1. [] Summary: Prefix skill (30min)
2. [x] DataStructure(Array):Diff skill + Binary search 

3. [x] SQL: (basic knowledge(filter / udf / subquery / standard join / stored procedure / transaction))

Nov. 16th . 2022
1. [x] DataStructure(Array):Sliding window

2. [x] 梳理一下 数据库 更加高级的 技能，（来源于康神的CMU 和 blog），Important!!! NOT MESS!!!
3. [] SQL

Nov. 17th . 2022
1. [x] 梳理 prefix skill / binary search / sliding window  

2. [x] CMU database 大学 ppt


Nov. 19th . 2022
1. [] 梳理 binary search 
2. [] 单调栈
3. [] 总结三篇 专业上的性能优化 
       （用SIMD指令优化**存储层**的热点代码｜向量化导入的优化｜查询的优化）
4. [] CMU database Multi-Version Concurrency Control 
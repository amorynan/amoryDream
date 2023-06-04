
### 一次版本在客户现场压测导致机器负载很高的排查过程
现象：

排查过程：（pstack / perf ）

结果解释和发生缘由：

char* -> wchar* 不统一处理 ==> try catch throw ==> 
clucen 大量trycatch ，finalize 处理释放的文件，回滚，加锁，

优化方案:

```sql
SELECT event_date, COUNT(DISTINCT m2) FROM ycs_activeDefense_v10_test_0118 WHERE event_date='2023-01-14'
UNION ALL
SELECT event_date, COUNT(DISTINCT m2) FROM  ycs_activeDefense_v10_test_0118 WHERE event_date='2023-01-15'
UNION ALL
SELECT event_date, COUNT(DISTINCT m2) FROM  ycs_activeDefense_v10_test_0118 WHERE event_date='2023-01-16'
UNION ALL 
SELECT event_date, COUNT(DISTINCT m2) FROM  ycs_activeDefense_v10_test_0118 WHERE event_date='2023-01-17'
UNION ALL 
SELECT event_date, COUNT(DISTINCT m2) FROM  ycs_activeDefense_v10_test_0118 WHERE event_date='2023-01-18'
UNION ALL 
SELECT event_date, COUNT(DISTINCT m2) FROM  ycs_activeDefense_v10_test_0118 WHERE event_date='2023-01-19'
ORDER BY event_date DESC
```
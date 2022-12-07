SESSION : Query Optimizer

SQL ->> QUERY-OPTIMIZER ->> PLAN(s) ->> MACHINE(multi core|mem|disk)

序：
    QUERY-OPTIMIZER 在👆的流程图可以看得出来，在数据库中扮演着 🧠 的位置，不管是OLAP， OLTP，HTAP 都会有不同实现方案，随着时间的推进，
一些优秀的思想和框架在时间的沉淀下，也慢慢占有他们的一席之地。本篇只是想以我浅薄的认知去以洞见一些优化器的本质。

1. SQL 的 何去何从?:
   ![img.png](../imgs/sqls.png)
   SQL 是给人看的，LogicPlan 是给 Optimizer 看的，Physical plan 是给 Executor 看的， 最后生成的机器码是给机器看的。

   SQL 从网络层Socket以二进制数据载体来了之后, DB 会拥有一个 SqlParser 类通过用当下比较流行的 ANTLR 插件生成对应的 Statement（ 
大体是经过 Lexer , Syntactic 形成一个 DB 中的类对象 Statement, eg.```SELECT * FROM test WHERE test.id = 1```   对应了 QueryStatement，涉及到SQL 编译原理，不展开）
同时，每一种Statement的相关语义上的check等操作，对于一些种类的Statement 会通过 StmtExecutor 生成对应种类的 ExecPlan. 之后就交给 StorageEngine 去执行, 重点来了
是否有一个最好的```执行计划```生成？

2. best ExecPlan (maybe ?!)的生成
   首先有必要预先普及一下一些名词，因为对于DB一些了解不深的人来说，看到这些词汇其实会比较摸不着头脑，就当作字典的开始目录吧，其次我会针对StarRocks 的source-code 进行一些概念的解读，
用于参照工业界的OLAP Optimizer 的实现
   ```sql
   Operator:(一种运算能力,关系代数的算子：https://zh.wikipedia.org/wiki/%E5%85%B3%E7%B3%BB%E4%BB%A3%E6%95%B0_(%E6%95%B0%E6%8D%AE%E5%BA%93))
    大体上我们常见分类有：
            "集合运算"：并集，差集，并集，笛卡尔积
            "投影 (π)"
            "选择 (σ)"
            "重命名 (ρ)"
            "自然连接 (⋈)": 两个关系的元组，组合条件是简单的共享属性上的相等
            "θ-连接和相等连接"：两个关系的元组，组合条件不是简单的共享属性上的相等，有< , <>
            "半连接 (⋉)(⋊)": 半连接的结果只是在S中有在公共属性名字上相等的元组所有的R中的元组
            "反连接 (▷)": 反连接的结果是在S中没有在公共属性名字上相等的元组的R中的那些元组。
            "左外连接 (⟕) | 右外连接 (⟖) | 全外连接 (⟗)"
            "聚集运算": AGG, MAX...
   
   ```
 
    

Velox(presto/spark):
DB2(OLTP):
StarRocks(OLAP):
ArangoDb(graph):
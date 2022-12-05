SESSION : Query Optimizer

SQL ->> QUERY-OPTIMIZER ->> PLAN(s) ->> MACHINE(multi core|mem|disk)

序：
    QUERY-OPTIMIZER 在👆的流程图可以看得出来，在数据库中扮演着 🧠 的位置，不管是OLAP， OLTP，HTAP 都会有不同实现方案，随着时间的推进，
一些优秀的思想和框架在时间的沉淀下，也慢慢占有他们的一席之地。本篇只是想以我浅薄的认知去以洞见一些优化器的本质。

1. SQL 的 何去何从:
    SQL 从网络层Socket以二进制数据载体来了之后, DB 会拥有一个 SqlParser 类通过用当下比较流行的 ANTLR 插件生成对应的 Statement（ 
大体是经过 Lexer , Syntactic 形成一个 DB 中的类对象 Statement, eg. SELECT * FROM TEST 对应了 QueryStatement，涉及到SQL 编译原理，不展开）
同时，每一种Statement的相关语义上的check等操作，对于一些种类的Statement 会通过 StmtExecutor 生成对应种类的 ExecPlan. 之后就交给 StorageEngine 去执行

2. ExecPlan 的生成
    
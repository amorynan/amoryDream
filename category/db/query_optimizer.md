SESSION : Query Optimizer

SQL ->> QUERY-OPTIMIZER ->> PLAN(s) ->> MACHINE(multi core|mem|disk)

序：
    QUERY-OPTIMIZER 在👆的流程图可以看得出来，在数据库中扮演着 🧠 的位置，不管是OLAP， OLTP，HTAP 都会有不同实现方案，随着时间的推进，
一些优秀的思想和框架在时间的沉淀下，也慢慢占有他们的一席之地。本篇只是想以我浅薄的认知去以洞见一些优化器的本质。

1. SQL 的 何去何从?:
   ![img.png](../imgs/sqls.png)
   SQL 是给人看的，LogicPlan 是给 Optimizer 看的，Physical plan 是给 Executor 看的， 最后生成的机器码是给机器看的。

   大致上工业界都是让 SQL 从网络层Socket以二进制数据载体来了之后, DB 会拥有一个 SqlParser 类通过用当下比较流行的 ANTLR 插件生成对应的 Statement（ 
大体是经过 Lexer（词法解析器） , Syntactic（语法解析器） 形成一个 DB 中的类对象 Statement, eg.```SELECT * FROM test WHERE test.id = 1```  被解析完就封装到了 QueryStatement class 对象中去，
这涉及到SQL 编译原理，不展开） 同时，每一种Statement的相关语义上的check等操作，那么问题就来了，Statement 的粒度在哪？拥有了很多不同操作粒度的Statement 之后咋交给机器看执行呢？说白了就是database如何```高效```
的和OS交流，才能让OS 将存储在memory 或者 cache 或者 disk(ssd/hdd) 吐给database，进而展示给用户看。
举一个具体的场景，QueryStatement 的生成, 自己感受下 ：

ANTLR 定义 ：(当我们写的sql 语句可以通过模式匹配到👇的一些组合的话，就在代码中可以通过 Parser 得到一个QueryStatement obj)
```mysql
explainDesc
: (DESC | DESCRIBE | EXPLAIN) (LOGICAL | VERBOSE | COSTS)?
    ;

queryStatement
    : explainDesc? queryBody outfile?;
    
queryBody
: withClause? queryNoWith
    ;
    
-- #with query 是一种在query 阶段定义的一种别名视图， 原始sql可以长这样
-- #-- define CTE:
-- # WITH Cost_by_Month AS
-- # (SELECT campaign_id AS campaign,
-- #        TO_CHAR(created_date, 'YYYY-MM') AS month,
-- #        SUM(cost) AS monthly_cost
-- # FROM marketing
-- # WHERE created_date BETWEEN NOW() - INTERVAL '3 MONTH' AND NOW()
-- # GROUP BY 1, 2
-- # ORDER BY 1, 2)
-- # 
-- # -- use CTE in subsequent query:
-- # SELECT campaign, avg(monthly_cost) as "Avg Monthly Cost"
-- # FROM Cost_by_Month
-- # GROUP BY campaign
-- # ORDER BY campaign
    
withClause
: WITH commonTableExpression (',' commonTableExpression)*
    ;

queryNoWith
:queryTerm (ORDER BY sortItem (',' sortItem)*)? (limitElement)?
    ;

queryTerm
: queryPrimary                                                             #queryTermDefault
| left=queryTerm operator=INTERSECT setQuantifier? right=queryTerm         #setOperation
| left=queryTerm operator=(UNION | EXCEPT | MINUS)
    setQuantifier? right=queryTerm                                         #setOperation
;

queryPrimary
: querySpecification                           #queryPrimaryDefault
| subquery                                     #subqueryPrimary
;

outfile
: INTO OUTFILE file=string fileFormat? properties?
    ;
```
```javascript
// parse to build query statement 
public ParseNode visitQueryStatement(StarRocksParser.QueryStatementContext context) {
    QueryRelation queryRelation = (QueryRelation) visit(context.queryBody());
    QueryStatement queryStatement = new QueryStatement(queryRelation);
    if (context.outfile() != null) {
        queryStatement.setOutFileClause((OutFileClause) visit(context.outfile()));
    }

    if (context.explainDesc() != null) {
        queryStatement.setIsExplain(true, getExplainType(context.explainDesc()));
    }

    return queryStatement;
}


public class QueryStatement extends StatementBase {
    private final QueryRelation queryRelation;

    // represent the "INTO OUTFILE" clause
    protected OutFileClause outFileClause;

    public QueryStatement(QueryRelation queryRelation) {
        this.queryRelation = queryRelation;
    }
```

   
2. best ExecPlan (maybe ?!)的生成
      首先有必要预先普及一下一些名词，因为对于DB一些了解不深的人来说，看到一些db人常说的词汇其实会比较摸不着头脑，就当作字典的开始目录吧，其次我会针对StarRocks 的source-code 进行一些概念的解读，
   用于参照工业界的OLAP Optimizer 的实现
      ```sql
      Tuple : wiki 上解释为从字段名到特定值的有限元素序列,eg. (name: amory, age: 25, cute: true) 作为一个tuple:表示25岁且可爱的amory 
      Relation Algebra(关系代数) : a set of fundamental operations to retrieve and manipulate tuples (检索和管理tuples 的一组操作集合)  
      
      Operator:(一种运算方式,关系代数的算子：https://zh.wikipedia.org/wiki/%E5%85%B3%E7%B3%BB%E4%BB%A3%E6%95%B0_(%E6%95%B0%E6%8D%AE%E5%BA%93))
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
               "标量运算": 标量函数将标量值（如整数或字符串）用作参数，并返回标量值作为结果。 可在 SQL 中任意可以传递标量值的地方使用标量函数, 如EXISTS, IN ...

      ```
      StarRocks 中定义的Operator , 大致有👇
      ![img.png](../imgs/operators.drawio.png)

```sql
    Expression: 关系代数的表达式，包括Operator ，在db 中其实我们经常把expression 看作一棵树的形式, 出自于我自己的感受，其一是树从某种
        程度上可以表示一定的顺序性，其二是在程序中递归树比起递归数组来说容易
        eg.[A ⋈ (B ⋈ C)](logical) : 表示 B 和 C 连接 再和 A 连接 
            ==> 如果指定一些物理上需要的具体操作，比如是merge join，hash join，还是nestloop join，就可以这样写
            [A(seq) ⋈ (NLJ) (B(idx) ⋈(HJ) C(seq))] : 表示 【索引scan B】 和 【顺序scan C】 以 【hash join 方式连接】 再和 【顺序scanA】 以 【nestloop join 连接】
    由⬆️可见[A ⋈ (B ⋈ C)]这样的一个式子其实就可以让机器按照我们想要的数据组合方式吐给我们了，也被称之为一个logic plan 和 physcal plan, 
        同时也可以明白，一个逻辑上的表达式可以和多个物理上的表达式相互对应，也就是一个逻辑上的plan 可以拥有多个物理上的plan, 响应而生就有
        Rule 概念的产生，因为Operator 算子也不是瞎转换的吧
```
    
众说周知，人与人说话都是一门艺术，嫁接到database，他与os 交流也是一门艺术哈哈哈（有点扩展了）

Velox(presto/spark):
DB2(OLTP):
StarRocks(OLAP):
ArangoDb(graph):
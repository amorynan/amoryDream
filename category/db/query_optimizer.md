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
        physical Op 是对于 logical Op 的一种具体算法的的实现， 比如 logical join 的 physical 实现有sort merge join 或者 hash join

```Query Tree: 至少包含一个logical operator 的查询计划树,他可以长这样```
![img.png](../imgs/col1.png)
```sql
 Execution plan: 通过一个具体的物理算子实现 query tree 上的 logical op的计划
```
![img_1.png](../imgs/col2.png)
```sql
    其实可以看到不管是 querytree 还是 execution plan 都是一种抽象的树，一个node + inputs(children),所以我们使用一个Expression
    Expression: 关系代数的表达式，包括Operator 和 这个Operator 的 inputs(也就是tree 的children)
```
StarRocks 中 OptExpression class 就是Expression 的表示
```javascript
/**
 * An expression is an operator with zero or more input expressions.
 * We refer to an expression as logical or physical
 * based on the type of its operator.
 * <p>
 * Logical Expression: (A ⨝ B) ⨝ C
 * Physical Expression: (AF ⨝HJ BF) ⨝NLJ CF
 */
public class OptExpression {
    // Expression 拥有的两个概念
    private Operator op;
    private List<OptExpression> inputs;
    
    //public class LogicalProperty {
        // Operator's output columns
        // private ColumnRefSet outputColumns;
        // The tablets num of left most scan node
        // private int leftMostScanTabletsNum;
        // The flag for execute upon less than or equal one tablet
        // private boolean isExecuteInOneTablet;
        //} // 保存一些当前operator 需要输出列的信息， 后面两个是后续算法优化需要的一些标志和额外信息
    
    private LogicalProperty property;
    private Statistics statistics;
    private double cost = 0;
    // The number of plans in the entire search space，this parameter is valid only when cbo_use_nth_exec_plan configured.
    // Default value is 0
    private int planCount = 0;
}
```
```sql  
不过新的问题很快就来了 (A ⨝ B) ⨝ C 和 (B ⨝ A) ⨝ C 这两个 OptExpression 看着它两 不相似莫
如果这两个logical OptExpression 的输出结果是一样的，那我们从逻辑上就判断他们是相等的，相等的OptExpression 用同一个表达式表示就是Group 
Group: logically equivalent expressions 
    就拿👆的两个OptExpression 举例，
    (A ⨝ B) 和 (B ⨝ A) 可以被归类到 [AB] Group 中
    ([AB] ⨝ C) 可以被归类到[ABC] Group 中
    其实就是把特定的排列，通过逻辑上的判断相同的排列们收集成一个组合， 这样做的收益可以多想想看
```
    
众说周知，人与人说话都是一门艺术，嫁接到database，他与os 交流也是一门艺术哈哈哈（有点扩展了）

Velox(presto/spark):
DB2(OLTP):
StarRocks(OLAP):
ArangoDb(graph):
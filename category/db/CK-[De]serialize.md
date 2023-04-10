![img.png](img.png)

CK 关于写入的调用栈分析

写入数据：
```cgo
DB::MySQLHandler::run()  MySQLHandler.cpp:176
DB::MySQLHandler::comQuery(ReadBuffer & payload)  MySQLHandler.cpp:367
DB::Interpreters::executeQuery(
    ReadBuffer & istr,
    WriteBuffer & ostr,
    bool allow_into_outfile,
    ContextMutablePtr context,
    SetResultDetailsFunc set_result_details,
    const std::optional<FormatSettings> & output_format_settings) Interpreters.cpp: 1260

static std::tuple<ASTPtr, BlockIO> executeQueryImpl(
    const char * begin,
    const char * end,
    ContextMutablePtr context,
    bool internal,
    QueryProcessingStage::Enum stage,
    ReadBuffer * istr) Interpreters.cpp:449
    
DB::InterpreterTransactionControlQuery::execute() 



```
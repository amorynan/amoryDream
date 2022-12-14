# SESSION : Storage Models, Data Layout, & System Catalogs

## Data Representation
1. simple data type :

2. complex data type : 
    
## Data Layout


## Complex data type design:
    design decisions:
        Tuple Identification
        Data Organization
        Update Policy
        Buffering Location

### Map

ClickHouse:

Map 复杂类型的支持， Map 80%场景需求：满足字段类型可变
1. 初始 #15806
   DataTypeMap -- 定义map 这个数据类型 序列化 和 反序列化
```cgo
// 大概有这些成员变量
using DataTypes = std::vector<DataTypePtr>;
private:
    DataTypePtr key_type;
    DataTypePtr value_type;
    DataTypePtr keys;
    DataTypePtr values;
    DataTypes kv;
// function：
    // c'tors
    DataTypeMap(const DataTypes & elems) {
        assert(elems_.size() < 3);
        key_type = elems_.size() == 1 ? DataTypeFactory::instance().get("String") : elems_[0];
        value_type = elems_.size() == 1 ? elems_[0] : elems_[1];
    
        keys = std::make_shared<DataTypeArray>(key_type);
        values = std::make_shared<DataTypeArray>(value_type);
        kv.push_back(keys); 
        kv.push_back(values);
    }
    
    const DataTypes & getElements() { return kv; }
    
    
```
 ColumnMap -- 定义列式存储的Map数据结构, 
```cgo
// 之前的版本
//using MapColumns = std::vector<WrappedPtr>;
// 现在的版本
/** Column, that is just group of few another columns.
  *
  * For constant Tuples, see ColumnConst.
  * Mixed constant/non-constant columns is prohibited in tuple
  *  for implementation simplicity.
  */
using TupleColumns = std::vector<WrappedPtr>;
    TupleColumns columns;
//private :
//    MapColumns columns;

// C'tors 之前的一次提交
//ColumnMap::ColumnMap(MutableColumns && mutable_columns)
//{
//    columns.reserve(mutable_columns.size());
//    for (auto & column : mutable_columns)
//    {
//        assert(column->getDataType() == TypeIndex::Array);
//        if (isColumnConst(*column))
//            throw Exception{"ColumnMap cannot have ColumnConst as its element", ErrorCodes::ILLEGAL_COLUMN};

//        columns.push_back(std::move(column));
//    }
//}
// 现在
ColumnTuple::Ptr ColumnTuple::create(const Columns & columns)
{
    for (const auto & column : columns)
        if (isColumnConst(*column))
            throw Exception{"ColumnTuple cannot have ColumnConst as its element", ErrorCodes::ILLEGAL_COLUMN};

    auto column_tuple = ColumnTuple::create(MutableColumns());
    column_tuple->columns.assign(columns.begin(), columns.end());

    return column_tuple;
}

// ====================  Map =====================
static Ptr create(const ColumnPtr & keys, const ColumnPtr & values, const ColumnPtr & offsets)
{
        // step1. ColumnTuple::create(Columns{keys, values})
        // step2. ColumnArray::create(ColumnTuple, offset)
        auto nested_column = ColumnArray::create(ColumnTuple::create(Columns{keys, values}), offsets);
        return ColumnMap::create(nested_column);
}

ColumnMap::ColumnMap(MutableColumnPtr && nested_)
    : nested(std::move(nested_))
{
    const auto * column_array = typeid_cast<const ColumnArray *>(nested.get());
    if (!column_array)
        throw Exception(ErrorCodes::LOGICAL_ERROR, "ColumnMap can be created only from array of tuples");

    const auto * column_tuple = typeid_cast<const ColumnTuple *>(column_array->getDataPtr().get());
    if (!column_tuple)
        throw Exception(ErrorCodes::LOGICAL_ERROR, "ColumnMap can be created only from array of tuples");

    if (column_tuple->getColumns().size() != 2)
        throw Exception(ErrorCodes::LOGICAL_ERROR, "ColumnMap should contain only 2 subcolumns: keys and values");

    for (const auto & column : column_tuple->getColumns())
        if (isColumnConst(*column))
            throw Exception(ErrorCodes::ILLEGAL_COLUMN, "ColumnMap cannot have ColumnConst as its element");
}

// resize is columnArray stuff
MutableColumnPtr ColumnMap::cloneResized(size_t new_size) const
{
    return ColumnMap::create(nested->cloneResized(new_size));
}

// get 
//void ColumnMap::get(size_t n, Field & res) const
//{
//    Map map(2);
    // columns[0] 中 get n-th to map[0]
//    columns[0]->get(n, map[0]);
//    columns[1]->get(n, map[1]);

//    res = map;
//} 

void ColumnMap::get(size_t n, Field & res) const
{
    //const ColumnArray & getNestedColumn() const { return assert_cast<const ColumnArray &>(*nested); }
    const auto & offsets = getNestedColumn().getOffsets();
    size_t offset = offsets[n - 1];
    size_t size = offsets[n] - offsets[n - 1];

    res = Map();
    auto & map = DB::get<Map &>(res);
    map.reserve(size);

    for (size_t i = 0; i < size; ++i)
        map.push_back(getNestedData()[offset + i]);
}

// insert 
//void ColumnMap::insert(const Field & x)
//{
//    const auto & map = DB::get<const Map &>(x);

//    if (map.size() != 2)
//        throw Exception("Cannot insert value of different size into map", ErrorCodes::CANNOT_INSERT_VALUE_OF_DIFFERENT_SIZE_INTO_TUPLE);

//    for (size_t i = 0; i < 2; ++i)
//        columns[i]->insert(map[i]);
//}

void ColumnMap::insert(const Field & x)
{
    const auto & map = DB::get<const Map &>(x);
    nested->insert(Array(map.begin(), map.end()));
    /**
    *size_t size = array.size();
    *for (size_t i = 0; i < size; ++i)
    *    getData().insert(array[i]);
    *getOffsets().push_back(getOffsets().back() + size);
    */
}

```





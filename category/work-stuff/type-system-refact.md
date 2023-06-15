## 数据类型重构 from_string
### 由于doris 目前在对于 text-based 的反序列化操作上不够灵活：
1. 不支持复杂类型的嵌套；
2. 不支持 text 的格式的设置；
3. Text 反序列化有比较多hard-code，不一定是对的（eg data_type_nullable 对于 NULL 的定义，array 类型对于next_element() 的定义）
4. from_string / to_string (二义性比较多， 比如 data_type 和 wrapField)
5. CSV 其实存在一部分内存的拷贝，性能上可能有影响
### Tried
   为了支持嵌套，尝试过用 from_json 去解析 text ，但是json 对于类型的定义其实还是会有比较多限制并且类json的本身类型比较少，对于一些doris 的类型，比如datetimev2 (2023-06-15 12:00:00) 这种只能在原始文本中先用 "" 代替，同时对于doris 类型系统来说对于后续开发无法做到扩展，比如支持用户自定义的格式，比如CK 的FormatSetting 设置 
### Better Thinking
   -- support format-setting like CK
      可以解决自定义解析文本格式的需求
   -- change from_string() to deserialize_column_from_text() with format setting
       可以解决嵌套类型反序列化
       可以解决去掉wrapField，使用deserialize_column_from_text()加上setting ，jingjing
### TODO Staff
   [x] Serde： 加上 deserialize_column_from_text(), 先不考虑 serialize_to_text() ，需要梳理整体doris中 to_string 的定义
   [x] function_cast 调用 data_type_to->from_string(readbuffer, column) 改成 data_type_to->get_serde()->deserialize_column_from_text(readerbuffer, column, format_setting)
   [x] 对于WrapperField 的优化思路：（需要UT 验证一下OlapField from/to_string VS data_type from/to_string 行为一致性，如果不一致，就在）
   
A. column_reader -> page_zoneMap (bytes(min, max))
   ->  (Min, Max)WrapperField.from_string()    ==> data_type->get_field(core)
   -> Field(Olap).from_string()                            ==> data_type->get_serde.deserialize_field_from_text(readBuffer, Field, setting)
   
B. column_mapping 替换 wrapperField 到 Field （default_value）
   从而 schema_change.cpp 也能改掉
   
C. 修改各种 xxx_predict中的WrapperField
1. BlockColumnPredicate evaluate_and()
2. ColumnPredicate evaluate_and() / evaluate_del()
   accep_null/comparison/in_list/null/bitmapFilter
### Check
   function_cast 支持 嵌套类型
   stream_load 支持嵌套输入，以及前后需要一个速度的对比（包含scala类型）
DataType
c++ 中根据编译器的数据模型所使用的数据类型所占有的bit 不一样

```cassandraql
Datetype  LP64   ILP64   LLP64   ILP32    LP32
char         8       8       8       8       8

short       16      16      16      16      16

int         32      64      32      32      16

long        64      64      32      32      32

long long   64

pointer     64      64      64      32      32
```
总体而言： 
char 在主流Windows， linux中不管是什么样的位数都是8bit, 1byte, 同理 short 是16bit， 2byte, int是32bit, 4byte
long 特殊在 LP64(linux/uinx) 是64 bit, 8byte, LLP64(windows) 是32bit, 4byte,这个分配和windows下int一致


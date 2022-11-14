1. seek for `1` in number
```go
// 消除 num 二进制中最后一个1
func eliminateLastOneInNumberBit(num int) {
	num & (num-1)
	return 
}
```
```go
// 求num 二进制中 1 的个数
func statisticalOneInNumberbit(num int) int {
	res := 0
	for num != 0 {
		res ++
		num = num & (num-1)
   }
}
```
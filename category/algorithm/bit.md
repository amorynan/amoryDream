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
```go
// 判断数组下标的存在性质： 判断 num(int) 值 二进制 x 位上是否有1
res = (num >> x) & 1 (res 是 bool值) 
// 设置 num(int) 某位x 上 为1 
res |= 1 << x
// 设置 num(int) 某位x 上 为0
res &^ = 1 << x
```
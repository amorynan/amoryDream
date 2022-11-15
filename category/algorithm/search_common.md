binary search func
```go
// common func for search in different situations, to find the smallest idx for satisfiedFunc == true
// witch means [0, idx-1) must satisfiedFunc=false
binarySearch(lth int, satisfiedFunc func(idx int) bool) int {
	l ,r := 0, lth-1
	for l < r {
	    m := int(uint(l+r)>>1) // avoid overflow 
		if !satisfiedFunc(m) {
			l = m+1 // to find next satisfied index
        }else {
			r = m // just narrow the range
        }
    }   
	return l
}

// how to use the interface up here
binarySearchFloatxxx(target floatxxx, arr []floatxxx) {
	idx := binarySearch(len(arr), func(idx int) bool { return arr[idx] > ? })
}
```
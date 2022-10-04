# 2. Program Structure
## 2.1. Names
### Case matters
名稱大小寫有差別。
heapSort != Heapsort
### Keywords
保留字，不能被當作 variable 跟 function 的名稱（識別字 identifier）。
- chan: 一種 data type，拿來宣告 channel 用。
- defer: 延遲 function 的執行 https://go.dev/tour/flowcontrol/12
- go: 用來開始 goroutine。
```
break        default      func         interface    select
case         defer*       go*          map          struct
chan*        else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```
### Predeclared identifiers
事先宣告的識別字，可以重新宣告。
```
Types:
	any bool byte comparable
	complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr

Constants:
	true false iota

Zero value:
	nil

Functions:
	append cap close complex copy delete imag len
	make new panic print println real recover
```
### Exported identifiers
Exported 代表可以從 package 之外的地方存取
滿足 Exported 的條件: 
1. 名稱第一個字母是大寫
2. 在 package block 裡有被定義或者是 field / method

field: go 語言裡面的 struct

method: interface 裡的 function

```
exported := false
if 名稱第一個字母 == 大寫 && (在 package block 裡有被定義 || isField || isMethod) {
	exported = true
}
```

### Uniqueness of identifiers
每個 identifier 都是不同的：
- 拼法不同
- 拼法相同但在不同 package 而且沒有 exported

換句話說：在拼法相同的前提下，如果是 exported 或 在同個 package 裡，就是相同的 identifier。
## 2.2. Declarations
## 2.3. Variables
## 2.4. Assignments
## 2.5. Type Declarations
## 2.6. Packages and Files
## 2.7. Scope
## Resource
- The Go Programming Language Specification: https://go.dev/ref/spec
- golang-study-group: https://github.com/sean1093/golang-study-group/blob/master/Notes/Chapter%202%20%E7%A8%8B%E5%BC%8F%E7%B5%90%E6%A7%8B.md
- The Go Programming Language: https://www.books.com.tw/products/F013508573

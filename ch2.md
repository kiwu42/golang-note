# 2. Program Structure
## 2.1. Names
### Naming Style
偏好駝峰式命名: camelCase 

縮寫會統一大小寫: 
- 建議的命名方式: htmlEscape、HTMLEscape 或 escapeHTML
- 反例: escapeHtml
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

```go
exported := false
if 名稱第一個字母 == 大寫 && (在 packageBlock 裡有被定義 || isField || isMethod) {
	exported = true
}
```

### Uniqueness of identifiers
每個 identifier 都是不同的：
- 拼法不同
- 拼法相同但在不同 package 而且沒有 exported

換句話說：在拼法相同的前提下，如果是 exported 或 在同個 package 裡，就是相同的 identifier。

## 2.2. Declarations
四種宣告種類: var, const, type, and func
```go
package main

import "fmt"

const boilingF = 212.0

func main() {
  var f = boilingF
  var c = (f - 32) * 5 / 9
  fmt.Printf("boiling point = %g F or %gC\n", f, c)
  // Output:
  // boiling point = 212 F or 100C
}
```
| Identifier | Declaration | kind |
| ----------- | ----------- | ----------- |
| boilingF | package-level | const |
| main | package-level | func |
| f, c | local | var |
```go
package main

import "fmt"

const boilingF = 212.0

func main() {
  const freezingF, boilingF = 32.0, 212.0

  fmt.Printf("%g F = %g C\n", freezingF, fToc(freezingF))
  fmt.Printf("%g F = %g C\n", boilingF, fToc(boilingF))
}

func fToc(f float64) float64 {
  return (f - 32) * 5 / 9
}
// Output:
// 32 F = 0 C
// 212 F = 100 C
```

> Q: package main 意思?
> 
> 一個 go 程式是由數個 package 組成，而 package 是由數個 source code (.go 檔)組成。
> 
> 在同個 package 裡可以共用 package-level 的 Identifier。
## 2.3. Variables
### Variable Declarations
```go
// general form
var name type = expression

// Omit type
var happy = "happy"
fmt.Printf("happy: %s\n", reflect.TypeOf(happy)) // happy: string

// Omit expression
var s string
fmt.Println(s) // ""

// Omit type and allow multiple varible declaration 
var i, j, k int // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```
### Zero-Value Mechanism
確保每個變數都有 well-defined 的值。
```go
package main

import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
	fmt.Printf("%v %v %v %q\n", i, f, b, s)
	// Output:
	// 0 0 false ""
	// array, struct: all elements hold zero value.
}
```
### Short Variable Declarations
在 function 內可以用 Short Variable Declarations 宣告並初始化 local variable。
```go
name := expression
```
Example:
```go
package main

import "fmt"

const boilingF = 212.0

func main() {
  f := boilingF
  c := (f - 32) * 5 / 9
  fmt.Printf("boiling point = %g F or %gC\n", f, c)
}
```
Special case:
```go
i, j = j, i // swap values of i and j
```
Note:
- 等號左邊可以有多個 varible
- 等號左邊必須要有一個新的 varible 被宣告
```go
f, err := os.Open(infile)
// ...
f, err := os.Create(outfile) // compile error: no new variables
```
```go
f, err := os.Open(infile)
// ...
f, err = os.Create(outfile) // fixxed
```
### Pointers
Pointers 儲存 variable 的位址。
```go
package main

import "fmt"

func main() {
	x := 1
	p := &x // p, of type *int, points to x's address
	fmt.Println(*p) // "1"
	*p = 2 // equivalent to x = 2
	fmt.Println(x) // "2"
}
```
每次呼叫 f() 都回傳不同的值( local variable 在 function 內被重新分配位址
```go
func f() *int {
  v := 1
  return &v
}
fmt.Println(f() == f()) // "false"
```

```go
func incr(p *int) int {
*p++ // increments what p points to; does not change p's value
return *p
}
v := 1
incr(&v) // side effect: v is now 2
fmt.Println(incr(&v)) // "3" (and v is 3)
```
flag package 就是用到 Pointer 的概念
```go
package main

import (
  "flag"
  "fmt"
  "strings"
)

var n = flag.Bool("n", false, "omit trailing newline")
var sep = flag.String("s", " ", "seperator")

func main() {
  flag.Parse() 
  fmt.Print(strings.Join(flag.Args(), *sep)) 

  if !*n {
    fmt.Println();
  }
}
```
```
Let’s run som e test cases on echo:
$ go build gopl.io/ch2/echo4
$ ./echo4 a bc def
a bc def
$ ./echo4 -s / a bc def(用/取代空白)
a/bc/def
$ ./echo4 -n a bc def(忽略換行)
a bc def$
$ ./echo4 -help
Usage of ./echo4:
-n omit trailing newline
-s string
separator (default " ")
```
### The new Function
Predeclared Function
1. 創一個沒命名的 variable(type 為 T)
2. 將 variable 初始化為 T 的零值
3. 回傳 T 的 address

```go
p := new(T)

//example
p := new(int) // p, of type *int, points to an unnamed int variable
fmt.Println(*p) // "0"
*p = 2 // sets the unnamed int to 2
fmt.Println(*p) // "2"
```
所以這兩個 function 其實是一樣的
```go
func newInt1() *int {
  return new(int)
}

func newInt2() *int {
  var dummy int
  return &dummy
}
```
幾乎每次呼叫 new 都會回傳不同的 address
除了一個特例：變數沒有定義，也就是說它的大小為零，如:struct{}, [0]int
```go
// Each call to new returnsadistinc t var iable wit h a unique address
p := new(int)
q := new(int)
fmt.Println(p == q) // "false"

// exception
arr1 := [1]int{0}
arr2 := [1]int{0}
fmt.Println(arr1 == arr2) // "true"
```
### Lifetime of Variables
在程式執行的時候，variable 存在的時間區間。
| Declaration | lifetime | default allocate |
| ----------- | ----------- | ----------- |
| package-level | 程式的執行時間 | heap |
| local | 動態安排(宣告到不再被引用) | stack |

不再被引用:unreachable

Note: Go 有 Garbage collection 的機制

## 2.4. Assignments
### Tuple Assignment
### Assignability
## 2.5. Type Declarations
## 2.6. Packages and Files
### Imports
### Package Initialization
## 2.7. Scope
## Resource
- The Go Programming Language Specification: https://go.dev/ref/spec
- golang-study-group: https://github.com/sean1093/golang-study-group/blob/master/Notes/Chapter%202%20%E7%A8%8B%E5%BC%8F%E7%B5%90%E6%A7%8B.md
- The Go Programming Language: https://www.books.com.tw/products/F013508573
- Markdown Cheat Sheet: https://www.markdownguide.org/cheat-sheet/

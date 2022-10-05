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
```go
x=1 // named variable
*p = true // indirect variable
person.name = "bob" // struct field

count[x] = count[x] * scale // array or slice or map element
count[x] *= scale

// Numeric variables can use ++ --
v := 1
v++ // same as v = v + 1; v becomes 2
v-- // same as v = v - 1; v becomes 1 again
```
### Tuple Assignment
一次更新多個等號左邊的 variable。
```go
x, y = y, x
a[i], a[j] = a[j], a[i]

func fib(n int) int {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		x, y = y, x+y
	}
	return x
}
```
等號右邊可以是一個回傳多個值的 function
```go
f, err = os.Open("foo.txt") // function call returns two values

v, ok = m[key] // map lookup
v, ok = x.(T) // type assertion
v, ok = <-ch // channel receive
```
用不到的值可以 assign 給佔位符(blank identifier)
```go
_, err = io.Copy(dst, src) // discard byte count
_, ok = x.(T) // check type but discard result
```
### Assignability
等號左右邊的型別要一樣才可以進行 assign。
```go
package main

func main() {
	y := 2
	y = "2"
	// Output:
	// error: cannot use "2" (untyped string constant) as int value in assignment
}
```
## 2.5. Type Declarations
Celsius(攝氏), Fahrenheit(華氏) 都是以 float64 為 underlying type 宣告的 type
- 攝氏不能跟華氏溫標來做運算比較 >> 避免error
- 需要重新定義轉換型別(conversion)的function(CToF and FToC)
- 如果直接呼叫 Celsius(t) 或 Fahrenheit(t)，會把 t 直接轉成 float64，t 的值不會變，不是我們要的結果
```go
// Package main performs Celsius and Fahrenheit temperature computations.
package main

import "fmt"

type Celsius float64
type Fahrenheit float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC     Celsius = 0
	BoilingC      Celsius = 100
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }

func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }
func main() {
	c := FToC(212.0)

	fmt.Println(c.String()) // 100°C
	fmt.Println("%v\n", c)  // 100°C
	fmt.Println("%s\n", c)  // 100°C
	fmt.Println(c)          // 100°C
	fmt.Println("%g\n", c)  // expect 100, but get "100°C"
	fmt.Println(float64(c)) // 100
}

```
## 2.6. Packages and Files
### Imports
import path: `gopl.io/ch2/tempconv`
package name:　tempconv
```go
// Cf converts its numeric argument to Celsius and Fahrenheit.
package main

import (
	"fmt"
	"os"
	"strconv"

	"gopl.io/ch2/tempconv"
)

func main() {
	for _, arg := range os.Args[1:] {
		t, err := strconv.ParseFloat(arg, 64)
		if err != nil {
			fmt.Fprintf(os.Stderr, "cf: %v\n", err)
			os.Exit(1)
		}
		f := tempconv.Fahrenheit(t)
		c := tempconv.Celsius(t)
		fmt.Printf("%s = %s, %s = %s\n",
			f, tempconv.FToC(f), c, tempconv.CToF(c))
	}
}
```

### Package Initialization
Package 初始化會依照宣告順序初始 package-level variables，除非有相依性。

如果 Package 有很多 .go檔，go tool 會先依照檔名排序後再叫 compiler。
```go
var a = b + c // a initialized third, to 3
var b = f() // b initialized second, to 2, by calling f
var c = 1 // c initialized first, to 1
func f() int { return c + 1 }
```
### init function

- 有些 variable 的初始化比較複雜，可以使用 init function
- init function 不能被呼叫或存取
- 當程式執行時，會自動依照 init functionS 被宣告的順序來一一執行它們。

```go
package popcount

// pc[i] is the population count of i.
var pc [256]byte

func init() {
	for i := range pc {
		pc[i] = pc[i/2] + byte(i&1)
	}
}

// PopCount returns the population count (number of set bits) of x.
func PopCount(x uint64) int {
	return int(pc[byte(x>>(0*8))] +
		pc[byte(x>>(1*8))] +
		pc[byte(x>>(2*8))] +
		pc[byte(x>>(3*8))] +
		pc[byte(x>>(4*8))] +
		pc[byte(x>>(5*8))] +
		pc[byte(x>>(6*8))] +
		pc[byte(x>>(7*8))])
}
```
### Q: package main 意思?

一個 go 程式是由數個 package 組成，而 package 是由數個 source code (.go 檔)組成。

在同個 package 裡可以共用 package-level 的 Identifier。

> 檔案結構：https://github.com/sean1093/golang-study-group/blob/master/Notes/Chapter%202%20%E7%A8%8B%E5%BC%8F%E7%B5%90%E6%A7%8B.md#6-%E5%A5%97%E4%BB%B6%E8%88%87%E6%AA%94%E6%A1%88

## 2.7. Scope
一個被宣告的 identfier 的 Scope，是指能存取 identfier 的程式碼片段。

### syntatic block

被{}包圍的區塊

### lexical block

就是 scope 

舉例來說:

| Declaration | scope |
| ----------- | ----------- |
| built-in type(len, int) |universe
block 整個程式都能用|
| package-level | 同個 package 裡的所有檔案 |
| file-level(import "fmt") | 有 import 的檔案|
| local | function 內的某一塊|
| flow control(break, continue) | for loop 之類的 |

當 identifier 命名不明確，會從最內層開始往外找。
```go
package main

import "fmt"

func f() {}

var g = "g"

func main() {
	f := "f"
	fmt.Println(f) // "f"; local var f shadows package-level func f
	fmt.Println(g) // "g"; package-level var
	fmt.Println(h) // compile error: undefined: h
}
```

### implicit blocks

for loops & if & switch 除了在 body 有可見的 blocks，也包含 implicit blocks

像是下面的程式 else if 裡除了 body 的fmt.Println(x, y)也可以存取第一個 if 宣告的 x
```go
if x := f(); x == 0 {
	fmt.Println(x)
} else if y := g(x); x == y {
	fmt.Println(x, y)
} else {
	fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```
### declaration order

package level 沒差

但 local 有差

舉例來說，f 的 scope 只在 if 裡面，所以下面超出 scope 的程式碼就無法存取 f。
```go
if f, err := os.Open(fname); err != nil { // compile error: unused: f
return err
}
f.ReadByte() // compile error: undefined f
f.Close() // compile error: undefined f
```
可以修改成這樣
```go
f, err := os.Open(fname)
if err != nil {
	return err
}
f.ReadByte()
f.Close()
```
Q: 阿為什麼不直接像下面用個else就好?

因為 GO 不喜歡你這樣做XD

GO 不建議縮排會被正常執行的code (Avoid nesting by handling errors first)

https://go.dev/talks/2013/bestpractices.slide#4

```go
if f, err := os.Open(fname); err != nil {
	return err
} else {
	// f and err are visible here too
	f.ReadByte()
	f.Close()
}
```
> 最後一個例子了!!!

cwd 被重新宣告成 local varible 了，結果 main 居然還沒用(那幹嘛宣告直接用_就好了嘛XDD
```go
package main

import (
	"log"
	"os"
)

var cwd string

func main() {
	cwd, err := os.Getwd() // cwd declared but not used

	if (err) != nil {
		log.Fatal("os.Getwd failed: %v", err)
	}
}
```
> 那如果我 main print 一下 cmd 是不是就用到了
```go
package main

import (
	"log" // 我把 os 去掉了XD
)

var cwd string = "我在這裡啊，你沒用到我還宣告我是怎樣"

func main() {
	cwd, err := "情況從糟糕變成難以理解", ""

	if (err) != "" {
		log.Fatal("os.Getwd failed: %v", err)
	}
	log.Printf("Working directory = %s", cwd)// 恩 果然沒跳error 不愧是我
	// 也是 Output: 情況從糟糕變成難以理解
}
```
可以看到我們 global 的`var cwd string = "我在這裡啊，你沒用到我還宣告我是怎樣"`是初始了個心酸

而且還沒跳error

真的是 情況從糟糕變成難以理解

沒事 我們可以這樣做

不要亂宣告(:=) 改成 assign(=)

就會用到我們預期要使用的 global cmd 了 是不是很酷
```go
package main

import (
  "log"
  "os"
)

var cwd string

func main() {
  var err error

  cwd, err = os.Getwd()

  if (err) != nil {
    log.Fatal("os.Getwd failed: %v", err)
  }
}
```

## Resource
- The Go Programming Language Specification: https://go.dev/ref/spec
- golang-study-group: https://github.com/sean1093/golang-study-group/blob/master/Notes/Chapter%202%20%E7%A8%8B%E5%BC%8F%E7%B5%90%E6%A7%8B.md
- The Go Programming Language: https://www.books.com.tw/products/F013508573
- Markdown Cheat Sheet: https://www.markdownguide.org/cheat-sheet/

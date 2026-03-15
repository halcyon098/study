

# Uber Go 风格指南

- [引言](#introduction)
- [规范](#guidelines)
  - [指向 interface 的指针](#pointers-to-interfaces)
  - [验证 interface 一致性](#verify-interface-compliance)
  - [接收器与 interface](#receivers-and-interfaces)
  - [零值 Mutex 可用](#zero-value-mutexes-are-valid)
  - [在边界处复制 slice 和 map](#copy-slices-and-maps-at-boundaries)
  - [使用 defer 做清理](#defer-to-clean-up)
  - [channel 容量为 1 或 0](#channel-size-is-one-or-none)
  - [枚举从 1 开始](#start-enums-at-one)
  - [用 "time" 处理时间](#use-time-to-handle-time)
  - [错误](#errors)
    - [错误类型](#error-types)
    - [错误包装](#error-wrapping)
    - [错误命名](#error-naming)
    - [一次性处理错误](#handle-errors-once)
  - [处理类型断言失败](#handle-type-assertion-failures)
  - [不要 panic](#dont-panic)
  - [使用 go.uber.org/atomic](#use-gouberorgatomic)
  - [避免可变全局变量](#avoid-mutable-globals)
  - [避免在公开 struct 中嵌入类型](#avoid-embedding-types-in-public-structs)
  - [避免使用内置名字](#avoid-using-built-in-names)
  - [避免 `init()`](#avoid-init)
  - [在 main 中退出](#exit-in-main)
    - [只退出一次](#exit-once)
  - [在序列化的 struct 中使用字段 tag](#use-field-tags-in-marshaled-structs)
  - [不要 fire-and-forget goroutine](#dont-fire-and-forget-goroutines)
    - [等待 goroutine 退出](#wait-for-goroutines-to-exit)
    - [`init()` 中不启动 goroutine](#no-goroutines-in-init)
- [性能](#performance)
  - [优先使用 strconv 而不是 fmt](#prefer-strconv-over-fmt)
  - [避免重复的 string→byte 转换](#avoid-repeated-string-to-byte-conversions)
  - [优先指定容器容量](#prefer-specifying-container-capacity)
- [风格](#style)
  - [避免过长的行](#avoid-overly-long-lines)
  - [保持一致](#be-consistent)
  - [相似声明分组](#group-similar-declarations)
  - [import 分组顺序](#import-group-ordering)
  - [包名](#package-names)
  - [函数名](#function-names)
  - [import 别名](#import-aliasing)
  - [函数分组与排序](#function-grouping-and-ordering)
  - [减少嵌套](#reduce-nesting)
  - [不必要的 else](#unnecessary-else)
  - [顶层变量声明](#top-level-variable-declarations)
  - [非导出全局变量以 _ 前缀](#prefix-unexported-globals-with-_)
  - [struct 中的嵌入](#embedding-in-structs)
  - [局部变量声明](#local-variable-declarations)
  - [nil 是合法的 slice](#nil-is-a-valid-slice)
  - [缩小变量作用域](#reduce-scope-of-variables)
  - [避免裸参数](#avoid-naked-parameters)
  - [使用原始字符串字面量避免转义](#use-raw-string-literals-to-avoid-escaping)
  - [struct 初始化](#initializing-structs)
    - [用字段名初始化 struct](#use-field-names-to-initialize-structs)
    - [在 struct 中省略零值字段](#omit-zero-value-fields-in-structs)
    - [零值 struct 使用 `var`](#use-var-for-zero-value-structs)
    - [struct 引用初始化](#initializing-struct-references)
  - [map 初始化](#initializing-maps)
  - [Printf 之外的格式字符串](#format-strings-outside-printf)
  - [Printf 风格函数命名](#naming-printf-style-functions)
- [模式](#patterns)
  - [测试表](#test-tables)
  - [函数式选项](#functional-options)
- [Linting](#linting)

## Introduction

风格是约束我们代码的约定。`style` 这个词有点名不副实，因为这些约定远不止源码格式——格式化由 `gofmt` 负责。

本指南的目标是通过详细说明在 Uber 编写 Go 代码的该做与不该做，来管理这种复杂度。这些规则让代码库保持可维护，同时仍然允许工程师高效使用 Go 语言特性。

本指南最初由 [Prashant Varanasi](https://github.com/prashantv) 和 [Simon Newton](https://github.com/nomis52) 编写，用来帮助同事快速上手 Go。多年来，它也根据他人的反馈不断补充完善。

本文档记录了 Uber 在 Go 代码中遵循的惯用约定。很多是 Go 的通用规范，也有一些是对外部资料的延伸：

1. [Effective Go](https://go.dev/doc/effective_go)
2. [Go Common Mistakes](https://go.dev/wiki/CommonMistakes)
3. [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)

我们的代码示例以 Go 最近两个小版本的 [releases](https://go.dev/doc/devel/release) 为基准，确保准确性。

所有代码在通过 `golint`（风格规范检查）和 `go vet`（静态分析）时应无错误。建议把编辑器配置为：

- 保存时运行 `goimports`（自动整理 import 并格式化）
- 运行 `golint`（风格规范检查）和 `go vet`（静态分析）检查错误

【2026 备注】`golint` 已废弃，建议使用 `golangci-lint`（包含 `staticcheck`）或直接用 `staticcheck`；`go vet` 仍推荐。

你可以在这里找到编辑器对 Go 工具的支持信息：
https://go.dev/wiki/IDEsAndTextEditorPlugins

## Guidelines

### Pointers to Interfaces

你几乎不需要指向 interface 的指针。应该以值形式传递 interface——底层数据依然可以是指针。

一个 interface 由两个字段组成：

1. 指向类型信息的指针，可理解为 "type"。
2. 数据指针。如果存储的是指针，会直接存进去；如果存储的是值，则会保存该值的指针。

如果你希望 interface 方法能够修改底层数据，就必须使用指针。

### Verify Interface Compliance

在合适的场景下做编译期的 interface 一致性验证，包括：

- 作为 API 契约要求实现特定 interface 的导出类型
- 某个类型集合中（导出或非导出）共同实现同一 interface 的类型
- 其它一旦破坏 interface 就会影响用户的情况

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Handler struct {
  // ...
}



func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  ...
}
```

</td><td>

```go
type Handler struct {
  // ...
}

var _ http.Handler = (*Handler)(nil)

func (h *Handler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

</td></tr>
</tbody></table>

语句 `var _ http.Handler = (*Handler)(nil)` 会在 `*Handler` 不再满足 `http.Handler` interface 时直接编译失败。

赋值右侧应该是被断言类型的零值。对于指针类型（如 `*Handler`）、slice 和 map，零值是 `nil`；对于 struct 类型，零值是空 struct。

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}

var _ http.Handler = LogHandler{}

func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

### Receivers and Interfaces

值接收器的方法既可以被指针调用，也可以被值调用。指针接收器的方法只能被指针或[可寻址值](https://go.dev/ref/spec#Method_values)调用。

例如，

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

// We cannot get pointers to values stored in maps, because they are not
// addressable values.
sVals := map[int]S{1: {"A"}}

// We can call Read on values stored in the map because Read
// has a value receiver, which does not require the value to
// be addressable.
sVals[1].Read()

// We cannot call Write on values stored in the map because Write
// has a pointer receiver, and it's not possible to get a pointer
// to a value stored in a map.
//
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// You can call both Read and Write if the map stores pointers,
// because pointers are intrinsically addressable.
sPtrs[1].Read()
sPtrs[1].Write("test")
```

同理，即便方法是值接收器，interface 也可以由指针来满足。

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// The following doesn't compile, since s2Val is a value, and there is no value receiver for f.
//   i = s2Val
```

Effective Go 对 [Pointers vs. Values](https://go.dev/doc/effective_go#pointers_vs_values) 有一段很好的说明。

### Zero-value Mutexes are Valid

`sync.Mutex` 和 `sync.RWMutex` 的零值是可用的，所以你几乎不需要指向 mutex 的指针。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

如果你用指针持有一个 struct，那么 mutex 应该是 struct 中的非指针字段。即使 struct 不导出，也不要把 mutex 直接嵌入到 struct 里。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type SMap struct {
  sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

<tr><td>

`Mutex` 字段以及 `Lock`、`Unlock` 方法意外地成为了 `SMap` 对外 API 的一部分。

</td><td>

mutex 及其方法属于 `SMap` 的实现细节，对调用方隐藏。

</td></tr>
</tbody></table>

### Copy Slices and Maps at Boundaries

slice 和 map 内部包含指向底层数据的指针，因此在需要复制时要格外小心。

#### Receiving Slices and Maps

注意：如果你保存了传入的 map 或 slice 的引用，调用方仍然可以修改它。

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// We can now modify trips[0] without affecting d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Returning Slices and Maps

同样要警惕：把 map 或 slice 直接返回给调用方可能暴露内部状态。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot returns the current stats.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot is no longer protected by the mutex, so any
// access to the snapshot is subject to data races.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Defer to Clean Up

使用 defer 来清理资源，比如文件和锁。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// easy to miss unlocks due to multiple returns
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
```

</td></tr>
</tbody></table>

`defer` 的开销极小，除非你能证明函数执行时间是纳秒级别，否则不应该为性能而避免使用它。`defer` 带来的可读性收益远大于这点微小成本。尤其是方法比较大、计算量远高于 `defer` 成本时，更是如此。

### Channel Size is One or None

channel 通常应当容量为 1 或者不带缓冲。默认情况下，channel 是无缓冲的，容量为 0。其它容量必须经过充分审视：容量如何确定？高负载下如何避免写阻塞？阻塞发生时系统如何表现？

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// Ought to be enough for anybody!
c := make(chan int, 64)
```

</td><td>

```go
// Size of one
c := make(chan int, 1) // or
// Unbuffered channel, size of zero
c := make(chan int)
```

</td></tr>
</tbody></table>

### Start Enums at One

Go 中引入枚举的标准方式是定义一个自定义类型，并用 `iota` 定义 `const` 组。因为变量默认值为 0，所以通常应该让枚举从非零值开始。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

也有使用零值更合适的场景，比如零值本身就是期望的默认行为。

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: section on String methods for enums -->

### Use "time" to handle time

时间很复杂，开发中经常出现以下错误假设：

1. 一天总是 24 小时
2. 一小时总是 60 分钟
3. 一周总是 7 天
4. 一年总是 365 天
5. [以及更多](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

例如，“加 24 小时一定是下一天”并不总成立。

因此，处理时间时始终使用 [`"time"`](https://pkg.go.dev/time) 包，它能更安全、更准确地处理这些错误假设。

#### Use `time.Time` for instants of time

处理时间点请使用 [`time.Time`](https://pkg.go.dev/time#Time)，比较、加减时间时用 `time.Time` 的方法。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}
```

</td><td>

```go
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

</td></tr>
</tbody></table>

#### Use `time.Duration` for periods of time

处理时间段请使用 [`time.Duration`](https://pkg.go.dev/time#Duration)。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // was it seconds or milliseconds?
```

</td><td>

```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

</td></tr>
</tbody></table>

回到“加 24 小时”的例子：该用哪种加法取决于意图。如果你想“同一时刻的下一自然日”，使用 [`Time.AddDate`](https://pkg.go.dev/time#Time.AddDate)。如果你想“严格 24 小时之后的时间点”，使用 [`Time.Add`](https://pkg.go.dev/time#Time.Add)。

```go
newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```

#### Use `time.Time` and `time.Duration` with external systems

与外部系统交互时，尽量使用 `time.Duration` 和 `time.Time`。例如：

- 命令行参数：[`flag`](https://pkg.go.dev/flag) 通过 [`time.ParseDuration`](https://pkg.go.dev/time#ParseDuration) 支持 `time.Duration`
- JSON：[`encoding/json`](https://pkg.go.dev/encoding/json) 通过 [`UnmarshalJSON` method](https://pkg.go.dev/time#Time.UnmarshalJSON) 支持将 `time.Time` 编码为 [RFC 3339](https://tools.ietf.org/html/rfc3339) 字符串
- SQL：[`database/sql`](https://pkg.go.dev/database/sql) 在驱动支持时，可在 `DATETIME`/`TIMESTAMP` 列与 `time.Time` 之间互转
- YAML：[`gopkg.in/yaml.v2`](https://pkg.go.dev/gopkg.in/yaml.v2) 支持将 `time.Time` 编码为 [RFC 3339](https://tools.ietf.org/html/rfc3339) 字符串，并通过 [`time.ParseDuration`](https://pkg.go.dev/time#ParseDuration) 支持 `time.Duration`

如果这些交互中无法使用 `time.Duration`，请用 `int` 或 `float64` 并在字段名里注明单位。

例如，由于 `encoding/json` 不支持 `time.Duration`，可以在字段名里注明单位：

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// {"interval": 2}
type Config struct {
  Interval int `json:"interval"`
}
```

</td><td>

```go
// {"intervalMillis": 2000}
type Config struct {
  IntervalMillis int `json:"intervalMillis"`
}
```

</td></tr>
</tbody></table>

如果这些交互中无法使用 `time.Time`，且没有明确的替代方案，则使用 `string` 并按 [RFC 3339](https://tools.ietf.org/html/rfc3339) 格式化时间戳。该格式也是 [`Time.UnmarshalText`](https://pkg.go.dev/time#Time.UnmarshalText) 的默认格式，并可通过 `Time.Format` 与 `time.Parse` 使用 [`time.RFC3339`](https://pkg.go.dev/time#RFC3339)。

虽然实践中通常不是问题，但要注意：`"time"` 包不支持解析带闰秒的时间戳（[8728](https://github.com/golang/go/issues/8728)），也不会在计算中考虑闰秒（[15190](https://github.com/golang/go/issues/15190)）。因此比较两个时间点时，差值不会包含期间发生的闰秒。

### Errors

#### Error Types

声明 error 有几种选择。选型前先想清楚：

- 调用方是否需要匹配 error 并做特定处理？如果需要，就必须支持 [`errors.Is`](https://pkg.go.dev/errors#Is) 或 [`errors.As`](https://pkg.go.dev/errors#As)，通过顶层 error 变量或自定义类型来实现。
- error 文案是静态字符串，还是需要上下文的动态字符串？前者用 [`errors.New`](https://pkg.go.dev/errors#New)，后者用 [`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf) 或自定义 error 类型。
- 是否只是传播下游函数返回的新 error？如果是，请看 [错误包装](#error-wrapping)。

| 是否需要匹配？ | 错误文案 | 建议 | 
|-----------------|---------|------|
| 否              | 静态    | [`errors.New`](https://pkg.go.dev/errors#New) |
| 否              | 动态    | [`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf) |
| 是              | 静态    | 顶层 `var` + [`errors.New`](https://pkg.go.dev/errors#New) |
| 是              | 动态    | 自定义 `error` 类型 |

例如：静态字符串的 error 用 [`errors.New`](https://pkg.go.dev/errors#New)。如果调用方需要匹配和处理该 error，就把它导出成变量，以便用 `errors.Is` 匹配。

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

if err := foo.Open(); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

对于动态字符串的 error：如果调用方不需要匹配，用 [`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf)；如果需要匹配，则用自定义 `error` 类型。

<table>
<thead><tr><th>No error matching</th><th>Error matching</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

// package bar

if err := foo.Open("testfile.txt"); err != nil {
  // Can't handle the error.
  panic("unknown error")
}
```

</td><td>

```go
// package foo

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

func Open(file string) error {
  return &NotFoundError{File: file}
}


// package bar

if err := foo.Open("testfile.txt"); err != nil {
  var notFound *NotFoundError
  if errors.As(err, &notFound) {
    // handle the error
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

注意：如果你从包中导出 error 变量或类型，它们将成为该包对外 API 的一部分。

#### Error Wrapping

调用失败后传播 error 有三种常见方式：

- 原样返回 error
- 用 `fmt.Errorf` + `%w` 增加上下文
- 用 `fmt.Errorf` + `%v` 增加上下文

如果没有额外上下文可加，就直接返回原始 error。这能保留原始 error 类型与信息，适合底层 error 信息足够定位来源的场景。

否则应尽量添加上下文，让错误从“connection refused”变成“call service foo: connection refused”这类更可行动的信息。

用 `fmt.Errorf` 添加上下文时，在 `%w` 与 `%v` 之间选择，取决于调用方是否需要匹配并提取底层原因：

- 若调用方需要访问底层 error，使用 `%w`。这是大多数包装 error 的默认选择，但要注意调用方可能会依赖这个行为。因此若被包装的 error 是已知 `var` 或类型，请在函数契约中写清并测试。
- 若想隐藏底层 error，则使用 `%v`。调用方无法匹配它，但你可以在未来改为 `%w`。

给返回的 error 增加上下文时，要避免“failed to”这种显而易见的前缀，因为 error 在栈上传播时会堆叠成啰嗦的文本：

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %w", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %w", err)
}
```

</td></tr><tr><td>

```plain
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```plain
x: y: new store: the error
```

</td></tr>
</tbody></table>

不过，当 error 被送到其它系统时，消息中应清楚地表明这是错误（例如日志中的 `err` 标签或 "Failed" 前缀）。

另见 [Don't just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)。

#### Error Naming

存放在全局变量中的 error，若导出用 `Err` 前缀，未导出用 `err` 前缀。这条规则优先级高于 [Prefix Unexported Globals with _](#prefix-unexported-globals-with-_)。

```go
var (
  // The following two errors are exported
  // so that users of this package can match them
  // with errors.Is.

  ErrBrokenLink = errors.New("link is broken")
  ErrCouldNotOpen = errors.New("could not open")

  // This error is not exported because
  // we don't want to make it part of our public API.
  // We may still use it inside the package
  // with errors.Is.

  errNotFound = errors.New("not found")
)
```

自定义 error 类型应使用 `Error` 作为后缀。

```go
// Similarly, this error is exported
// so that users of this package can match it
// with errors.As.

type NotFoundError struct {
  File string
}

func (e *NotFoundError) Error() string {
  return fmt.Sprintf("file %q not found", e.File)
}

// And this error is not exported because
// we don't want to make it part of the public API.
// We can still use it inside the package
// with errors.As.

type resolveError struct {
  Path string
}

func (e *resolveError) Error() string {
  return fmt.Sprintf("resolve %q", e.Path)
}
```

#### Handle Errors Once

调用方从被调用方拿到 error 后，可以根据掌握的信息用不同方式处理，例如：

- 如果被调用方在契约中定义了特定 error，使用 `errors.Is` 或 `errors.As` 匹配后分支处理
- 如果 error 可恢复，记录日志并降级处理
- 如果 error 表示领域内的失败条件，返回一个明确的 error
- 直接返回 error（[包装](#error-wrapping) 或原样返回）

无论调用方如何处理 error，通常都应“只处理一次”。比如不要在记录日志后再返回 error，否则上层调用方可能再次处理，导致日志噪音。

例如：

<table>
<thead><tr><th>Description</th><th>Code</th></tr></thead>
<tbody>
<tr><td>

**Bad**：记录日志后返回 error

栈上更上层的调用方很可能也会做同样的事，这会让日志非常嘈杂而收益很低。

</td><td>

```go
u, err := getUser(id)
if err != nil {
  // BAD: See description
  log.Printf("Could not get user %q: %v", id, err)
  return err
}
```

</td></tr>
<tr><td>

**Good**：包装后返回 error

上层调用方负责处理 error。使用 `%w` 使其可通过 `errors.Is` 或 `errors.As` 进行匹配（如有需要）。

</td><td>

```go
u, err := getUser(id)
if err != nil {
  return fmt.Errorf("get user %q: %w", id, err)
}
```

</td></tr>
<tr><td>

**Good**：记录日志并降级

当操作并非强依赖时，可以通过恢复来提供降级但不中断的体验。

</td><td>

```go
if err := emitMetrics(); err != nil {
  // Failure to write metrics should not
  // break the application.
  log.Printf("Could not emit metrics: %v", err)
}

```

</td></tr>
<tr><td>

**Good**：匹配 error 并降级

如果被调用方契约中定义了特定 error 且该失败可恢复，就匹配该 error 并降级。其他情况则包装并返回。

上层调用方会处理其它 error。

</td><td>

```go
tz, err := getUserTimeZone(id)
if err != nil {
  if errors.Is(err, ErrUserNotFound) {
    // User doesn't exist. Use UTC.
    tz = time.UTC
  } else {
    return fmt.Errorf("get user %q: %w", id, err)
  }
}
```

</td></tr>
</tbody></table>

### Handle Type Assertion Failures

单返回值的 [type assertion](https://go.dev/ref/spec#Type_assertions) 在类型不匹配时会 panic，所以务必使用 "comma ok" 习惯用法。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->

### Don't Panic

生产环境运行的代码必须避免 panic。panic 是 [级联故障](https://en.wikipedia.org/wiki/Cascading_failure) 的重要来源之一。出现错误时应返回 error，把如何处理交给调用方。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func run(args []string) {
  if len(args) == 0 {
    panic("an argument is required")
  }
  // ...
}

func main() {
  run(os.Args[1:])
}
```

</td><td>

```go
func run(args []string) error {
  if len(args) == 0 {
    return errors.New("an argument is required")
  }
  // ...
  return nil
}

func main() {
  if err := run(os.Args[1:]); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```

</td></tr>
</tbody></table>

panic/recover 不是错误处理策略。程序只能在不可恢复的情况（例如 nil 解引用）下 panic。一个例外是程序初始化：启动阶段出现致命问题可以选择 panic。

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

即便在测试里，也应优先使用 `t.Fatal` 或 `t.FailNow`，确保测试被标记为失败，而不是 panic。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := os.CreateTemp("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

### Use go.uber.org/atomic

使用 [sync/atomic](https://pkg.go.dev/sync/atomic) 做原子操作时，操作的是基础类型（`int32`、`int64` 等），很容易忘记用原子操作来读写变量。

[go.uber.org/atomic](https://pkg.go.dev/go.uber.org/atomic) 通过隐藏底层类型提供了类型安全，同时还包含方便的 `atomic.Bool`。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>

### Avoid Mutable Globals

避免修改全局变量，优先使用依赖注入。这不仅适用于函数指针，也适用于其他类型的值。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// sign.go

var _timeNow = time.Now

func sign(msg string) string {
  now := _timeNow()
  return signWithTime(msg, now)
}
```

</td><td>

```go
// sign.go

type signer struct {
  now func() time.Time
}

func newSigner() *signer {
  return &signer{
    now: time.Now,
  }
}

func (s *signer) Sign(msg string) string {
  now := s.now()
  return signWithTime(msg, now)
}
```

</td></tr>
<tr><td>

```go
// sign_test.go

func TestSign(t *testing.T) {
  oldTimeNow := _timeNow
  _timeNow = func() time.Time {
    return someFixedTime
  }
  defer func() { _timeNow = oldTimeNow }()

  assert.Equal(t, want, sign(give))
}
```

</td><td>

```go
// sign_test.go

func TestSigner(t *testing.T) {
  s := newSigner()
  s.now = func() time.Time {
    return someFixedTime
  }

  assert.Equal(t, want, s.Sign(give))
}
```

</td></tr>
</tbody></table>

### Avoid Embedding Types in Public Structs

嵌入类型会泄露实现细节、限制类型演进，并让文档变得含糊。

假设你通过一个共享的 `AbstractList` 实现了多种列表类型，那么应避免在具体列表实现中嵌入 `AbstractList`。相反，手写这些具体列表中需要的委托方法。

```go
type AbstractList struct {}

// Add adds an entity to the list.
func (l *AbstractList) Add(e Entity) {
  // ...
}

// Remove removes an entity from the list.
func (l *AbstractList) Remove(e Entity) {
  // ...
}
```

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  *AbstractList
}
```

</td><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  list *AbstractList
}

// Add adds an entity to the list.
func (l *ConcreteList) Add(e Entity) {
  l.list.Add(e)
}

// Remove removes an entity from the list.
func (l *ConcreteList) Remove(e Entity) {
  l.list.Remove(e)
}
```

</td></tr>
</tbody></table>

Go 允许 [type embedding](https://go.dev/doc/effective_go#embedding)，作为继承与组合之间的折中。外层类型会隐式获得内嵌类型的方法副本，这些方法默认委托给内嵌实例的同名方法。

struct 也会获得一个与内嵌类型同名的字段。所以如果内嵌类型是公开的，该字段也是公开的。为了保持向后兼容，外层类型的每个未来版本都必须保留该内嵌类型。

内嵌类型很少是必要的，它只是让你避免写繁琐的委托方法的便利手段。

即便内嵌的是一个兼容的 AbstractList interface，而不是 struct，也会给未来演进更多弹性，但仍然泄露了“具体列表使用抽象实现”这一细节。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// AbstractList is a generalized implementation
// for various kinds of lists of entities.
type AbstractList interface {
  Add(Entity)
  Remove(Entity)
}

// ConcreteList is a list of entities.
type ConcreteList struct {
  AbstractList
}
```

</td><td>

```go
// ConcreteList is a list of entities.
type ConcreteList struct {
  list AbstractList
}

// Add adds an entity to the list.
func (l *ConcreteList) Add(e Entity) {
  l.list.Add(e)
}

// Remove removes an entity from the list.
func (l *ConcreteList) Remove(e Entity) {
  l.list.Remove(e)
}
```

</td></tr>
</tbody></table>

无论内嵌 struct 还是 interface，都会限制类型演进：

- 给内嵌 interface 新增方法是破坏性变更。
- 从内嵌 struct 移除方法是破坏性变更。
- 移除内嵌类型是破坏性变更。
- 替换内嵌类型，即便新类型满足同一 interface，也是破坏性变更。

虽然手写委托方法比较繁琐，但它隐藏了实现细节，给未来演进留出空间，也避免了在文档中寻找完整 List interface 时的间接性。

### Avoid Using Built-In Names

Go [language specification](https://go.dev/ref/spec) 定义了若干内置的 [predeclared identifiers](https://go.dev/ref/spec#Predeclared_identifiers)，不应在 Go 程序中用作命名。

视上下文而定，重复使用这些标识符会在当前词法作用域（及其嵌套作用域）中遮蔽原始标识符，或让代码难以理解。最好情况下编译器会报错；最坏情况下可能引入隐蔽、难以检索的 bug。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var error string
// `error` shadows the builtin

// or

func handleErrorMessage(error string) {
    // `error` shadows the builtin
}
```

</td><td>

```go
var errorMessage string
// `error` refers to the builtin

// or

func handleErrorMessage(msg string) {
    // `error` refers to the builtin
}
```

</td></tr>
<tr><td>

```go
type Foo struct {
    // While these fields technically don't
    // constitute shadowing, grepping for
    // `error` or `string` strings is now
    // ambiguous.
    error  error
    string string
}

func (f Foo) Error() error {
    // `error` and `f.error` are
    // visually similar
    return f.error
}

func (f Foo) String() string {
    // `string` and `f.string` are
    // visually similar
    return f.string
}
```

</td><td>

```go
type Foo struct {
    // `error` and `string` strings are
    // now unambiguous.
    err error
    str string
}

func (f Foo) Error() error {
    return f.err
}

func (f Foo) String() string {
    return f.str
}
```

</td></tr>
</tbody></table>

注意：编译器不会因为使用 predeclared identifiers 而报错，但 `go vet`（静态分析）等工具应能正确指出这类遮蔽问题。

### Avoid `init()`

尽量避免 `init()`。如果必须使用 `init()`，代码应尽量：

1. 完全确定性，与程序环境或调用方式无关。
2. 不依赖其它 `init()` 的执行顺序或副作用。尽管 `init()` 顺序是已知的，但代码会变化，`init()` 之间的关系会让代码脆弱且易错。
3. 避免访问或操作全局或环境状态，例如机器信息、环境变量、工作目录、程序参数/输入等。
4. 避免 I/O，包括文件系统、网络和系统调用。

无法满足这些要求的逻辑通常应该作为 `main()`（或生命周期中的其它阶段）调用的辅助函数，或直接写入 `main()` 本身。特别是给其它程序使用的库，更要保证完全确定性，不做 "init magic"。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Foo struct {
    // ...
}

var _defaultFoo Foo

func init() {
    _defaultFoo = Foo{
        // ...
    }
}
```

</td><td>

```go
var _defaultFoo = Foo{
    // ...
}

// or, better, for testability:

var _defaultFoo = defaultFoo()

func defaultFoo() Foo {
    return Foo{
        // ...
    }
}
```

</td></tr>
<tr><td>

```go
type Config struct {
    // ...
}

var _config Config

func init() {
    // Bad: based on current directory
    cwd, _ := os.Getwd()

    // Bad: I/O
    raw, _ := os.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )

    yaml.Unmarshal(raw, &_config)
}
```

</td><td>

```go
type Config struct {
    // ...
}

func loadConfig() Config {
    cwd, err := os.Getwd()
    // handle err

    raw, err := os.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )
    // handle err

    var config Config
    yaml.Unmarshal(raw, &config)

    return config
}
```

</td></tr>
</tbody></table>

综合上面的要求，`init()` 更适合或必要的情况包括：

- 复杂表达式无法用单行赋值表示。
- 可插拔的 hook，比如 `database/sql` 方言、编码类型注册表等。
- 针对 [Google Cloud Functions](https://cloud.google.com/functions/docs/bestpractices/tips#use_global_variables_to_reuse_objects_in_future_invocations) 的优化，以及其它形式的确定性预计算。

### Exit in Main

Go 程序使用 [`os.Exit`](https://pkg.go.dev/os#Exit) 或 [`log.Fatal*`](https://pkg.go.dev/log#Fatal) 立即退出。（panic 不是退出程序的好方法，请 [don't panic](#dont-panic)。）

只在 `main()` 中调用 `os.Exit` 或 `log.Fatal*`。其它函数应该返回 error 来传递失败。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func main() {
  body := readFile(path)
  fmt.Println(body)
}

func readFile(path string) string {
  f, err := os.Open(path)
  if err != nil {
    log.Fatal(err)
  }

  b, err := io.ReadAll(f)
  if err != nil {
    log.Fatal(err)
  }

  return string(b)
}
```

</td><td>

```go
func main() {
  body, err := readFile(path)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(body)
}

func readFile(path string) (string, error) {
  f, err := os.Open(path)
  if err != nil {
    return "", err
  }

  b, err := io.ReadAll(f)
  if err != nil {
    return "", err
  }

  return string(b), nil
}
```

</td></tr>
</tbody></table>

理由：多个函数可能退出程序会带来一些问题：

- 控制流不清晰：任何函数都可能直接退出，难以推理执行路径。
- 测试困难：退出程序的函数会直接退出调用它的测试，导致测试无法继续执行。
- 清理被跳过：退出会跳过已通过 `defer` 注册的清理逻辑，增加资源泄露风险。

#### Exit Once

如果可以，尽量在 `main()` 中 **最多只调用一次** `os.Exit` 或 `log.Fatal`。如果有多个会终止执行的错误场景，把逻辑放在单独函数里返回 error。

这样 `main()` 更短，核心业务逻辑集中在可测试的函数中。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
package main

func main() {
  args := os.Args[1:]
  if len(args) != 1 {
    log.Fatal("missing file")
  }
  name := args[0]

  f, err := os.Open(name)
  if err != nil {
    log.Fatal(err)
  }
  defer f.Close()

  // If we call log.Fatal after this line,
  // f.Close will not be called.

  b, err := io.ReadAll(f)
  if err != nil {
    log.Fatal(err)
  }

  // ...
}
```

</td><td>

```go
package main

func main() {
  if err := run(); err != nil {
    log.Fatal(err)
  }
}

func run() error {
  args := os.Args[1:]
  if len(args) != 1 {
    return errors.New("missing file")
  }
  name := args[0]

  f, err := os.Open(name)
  if err != nil {
    return err
  }
  defer f.Close()

  b, err := io.ReadAll(f)
  if err != nil {
    return err
  }

  // ...
}
```

</td></tr>
</tbody></table>

上面的示例使用了 `log.Fatal`，但同样适用于 `os.Exit` 或任何调用了 `os.Exit` 的库代码。

```go
func main() {
  if err := run(); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```

你可以按需要调整 `run()` 的签名。例如，如果程序需要返回特定退出码，`run()` 可以返回退出码而不是 error，这样单测也更易验证。

```go
func main() {
  os.Exit(run(args))
}

func run() (exitCode int) {
  // ...
}
```

更一般地说，这些示例中的 `run()` 并不具有强制性。名称、签名和初始化方式都有弹性，例如：

- 接收未解析的命令行参数（如 `run(os.Args[1:])`）
- 在 `main()` 解析参数后再传给 `run`
- 使用自定义 error 类型携带退出码回到 `main()`
- 将业务逻辑放在 `package main` 之外的层次

本规范只要求：在 `main()` 中有一个唯一的地方负责真正退出进程。

### Use field tags in marshaled structs

任何会被序列化到 JSON、YAML 或其他支持 tag 命名字段的格式中的 struct 字段，都应标注相应的 tag。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stock struct {
  Price int
  Name  string
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "UBER",
})
```

</td><td>

```go
type Stock struct {
  Price int    `json:"price"`
  Name  string `json:"name"`
  // Safe to rename Name to Symbol.
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "UBER",
})
```

</td></tr>
</tbody></table>

理由：序列化后的结构是系统间的契约。对序列化结构的更改——包括字段名——都会破坏契约。通过 tag 指定字段名能显式表达契约，并防止重构/改名时无意破坏契约。

### Don't fire-and-forget goroutines

goroutine 很轻量，但不是免费的：至少会占用栈内存，并消耗 CPU 调度。对一般场景成本不大，但如果大量创建且缺乏生命周期管理，会造成明显性能问题。生命周期不可控的 goroutine 还可能阻止无用对象被 GC 回收，或长期占用已不再需要的资源。

因此，生产代码中不要泄漏 goroutine。对可能启动 goroutine 的包，使用 [go.uber.org/goleak](https://pkg.go.dev/go.uber.org/goleak) 测试 goroutine 泄漏。

总体来说，每个 goroutine：

- 必须有可预期的停止时间；或
- 必须有办法发信号让它停止

无论哪种情况，都要有办法阻塞等待该 goroutine 结束。

例如：

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
go func() {
  for {
    flush()
    time.Sleep(delay)
  }
}()
```

</td><td>

```go
var (
  stop = make(chan struct{}) // tells the goroutine to stop
  done = make(chan struct{}) // tells us that the goroutine exited
)
go func() {
  defer close(done)

  ticker := time.NewTicker(delay)
  defer ticker.Stop()
  for {
    select {
    case <-ticker.C:
      flush()
    case <-stop:
      return
    }
  }
}()

// Elsewhere...
close(stop)  // signal the goroutine to stop
<-done       // and wait for it to exit
```

</td></tr>
<tr><td>

这个 goroutine 无法停止，只能一直跑到应用退出。

</td><td>

这个 goroutine 可以通过 `close(stop)` 停止，也可以通过 `<-done` 等待退出。

</td></tr>
</tbody></table>

#### Wait for goroutines to exit

如果系统启动了 goroutine，必须有办法等待它结束。常见做法有两种：

- 使用 `sync.WaitGroup` 等待多个 goroutine 完成（适用于多个 goroutine 需要等待的场景）。

  ```go
  var wg sync.WaitGroup
  for i := 0; i < N; i++ {
    wg.Go(...)
  }

  // To wait for all to finish:
  wg.Wait()
  ```

- 增加一个 `chan struct{}`，让 goroutine 结束时关闭它（适用于只有一个 goroutine 的场景）。

  ```go
  done := make(chan struct{})
  go func() {
    defer close(done)
    // ...
  }()
  
  // To wait for the goroutine to finish:
  <-done
  ```

#### No goroutines in `init()`

`init()` 不应该启动 goroutine。另见 [Avoid init()](#avoid-init)。

如果某个包需要后台 goroutine，必须暴露一个负责管理 goroutine 生命周期的对象，并提供方法（`Close`、`Stop`、`Shutdown` 等）来通知 goroutine 停止并等待退出。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func init() {
  go doWork()
}

func doWork() {
  for {
    // ...
  }
}
```

</td><td>

```go
type Worker struct{ /* ... */ }

func NewWorker(...) *Worker {
  w := &Worker{
    stop: make(chan struct{}),
    done: make(chan struct{}),
    // ...
  }
  go w.doWork()
  return w
}

func (w *Worker) doWork() {
  defer close(w.done)
  for {
    // ...
    case <-w.stop:
      return
  }
}

// Shutdown tells the worker to stop
// and waits until it has finished.
func (w *Worker) Shutdown() {
  close(w.stop)
  <-w.done
}
```

</td></tr>
<tr><td>

当用户导入该包时就无条件启动后台 goroutine。用户既无法控制该 goroutine，也无法停止它。

</td><td>

只有在用户显式请求时才启动 worker。提供关闭 worker 的手段，方便用户释放资源。

注意：如果 worker 管理多个 goroutine，请使用 `WaitGroup`。参见 [Wait for goroutines to exit](#wait-for-goroutines-to-exit)。

</td></tr>
</tbody></table>

## Performance

只在热点路径上应用性能相关规范。

### Prefer strconv over fmt

在原始类型与字符串之间转换时，`strconv` 比 `fmt` 更快。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```plain
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```plain
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Avoid repeated string-to-byte conversions

不要反复从固定字符串创建 `[]byte`。应只转换一次并复用结果。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</td></tr>
<tr><td>

```plain
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```plain
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

### Prefer Specifying Container Capacity

尽量在创建容器时指定容量，以便一次性分配内存，减少后续因为扩容带来的复制和重新分配。

#### Specifying Map Capacity Hints

初始化 map 时尽量给 `make()` 提供容量提示。

```go
make(map[T1]T2, hint)
```

给 `make()` 提示容量会在初始化阶段尽量把 map 调到合适大小，从而减少后续扩容与分配。

注意：与 slice 不同，map 的容量提示并不保证完整、一次性分配，它只是用于估算需要的 bucket 数。因此即便在容量范围内插入元素，也可能产生分配。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
files, _ := os.ReadDir("./files")

m := make(map[string]os.DirEntry)
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := os.ReadDir("./files")

m := make(map[string]os.DirEntry, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` 创建时没有容量提示，map 会动态扩容，导致多次分配。

</td><td>

`m` 创建时提供了容量提示，插入时可能减少分配。

</td></tr>
</tbody></table>

#### Specifying Slice Capacity

初始化 slice（尤其是后续要 append）时，尽量提供容量提示。

```go
make([]T, length, capacity)
```

与 map 不同，slice 的容量不是提示：编译器会按 `make()` 中的容量直接分配内存，意味着后续 `append()` 在容量耗尽前不会产生分配（直到长度达到容量，之后才会扩容）。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, 0)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, 0, size)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td></tr>
<tr><td>

```plain
BenchmarkBad-4    100000000    2.48s
```

</td><td>

```plain
BenchmarkGood-4   100000000    0.21s
```

</td></tr>
</tbody></table>

## Style

### Avoid overly long lines

避免需要读者横向滚动或歪头阅读的长行。

我们建议软性行宽上限为 **99 字符**。作者应在接近该上限前换行，但这不是硬性限制，允许偶尔超出。

### Be Consistent

本文部分规范可以客观评估，部分则取决于场景、上下文或主观判断。

最重要的是：**保持一致**。

一致的代码更易维护、更容易理解迁移，认知成本更低。随着新规范或新类 bug 被修复，一致性也让演进更容易。

相反，同一个代码库里存在多种互相冲突或分散的风格，会带来维护成本、不确定性和认知割裂，直接拖慢研发速度、增加痛苦的代码评审并引入 bug。

应用这些规范时，建议在包级（或更大的范围）统一执行：在子包级别执行会把多种风格引入同一代码库，违背上述原则。

### Group Similar Declarations

Go 支持将相似声明分组。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

这同样适用于常量、变量和类型声明。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

只对相关声明分组，不要把不相关的声明放在同一组。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  EnvVar = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const EnvVar = "MY_ENV"
```

</td></tr>
</tbody></table>

分组不受使用位置限制，例如可以在函数内部使用。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  red := color.New(0xff0000)
  green := color.New(0x00ff00)
  blue := color.New(0x0000ff)

  // ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  // ...
}
```

</td></tr>
</tbody></table>

例外：变量声明（尤其在函数内部）如果彼此相邻，应被分组在一起，即便它们之间不相关。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (c *client) request() {
  caller := c.name
  format := "json"
  timeout := 5*time.Second
  var err error

  // ...
}
```

</td><td>

```go
func (c *client) request() {
  var (
    caller  = c.name
    format  = "json"
    timeout = 5*time.Second
    err error
  )

  // ...
}
```

</td></tr>
</tbody></table>

### Import Group Ordering

import 分组应当只有两组：

- 标准库
- 其它所有内容

这是 `goimports` 默认的分组方式。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Package Names

包命名应满足：

- 全小写，不用大写或下划线。
- 在大多数调用点不需要通过别名重命名。
- 简短、精炼。注意包名会在每个调用点完整出现。
- 不用复数形式，例如 `net/url` 而不是 `net/urls`。
- 不要用 "common"、"util"、"shared" 或 "lib" 等模糊且无信息量的名称。

另见 [Package Names](https://go.dev/blog/package-names) 与 [Style guideline for Go packages](https://rakyll.org/style-packages/)。

### Function Names

遵循 Go 社区约定，函数名使用 [MixedCaps](https://go.dev/doc/effective_go#mixed-caps)。测试函数是例外，为了分组相关用例，可以包含下划线，例如 `TestMyFunction_WhatIsBeingTested`。

### Import Aliasing

当包名与 import 路径最后一段不一致时，必须使用别名。

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

其他情况下应尽量避免使用别名，除非 import 之间有直接冲突。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  runtimetrace "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

### Function Grouping and Ordering

- 函数应按大致调用顺序排序。
- 同一文件内的函数应按接收器分组。

因此，导出函数应出现在文件前部，且位于 `struct`、`const`、`var` 定义之后。

`newXYZ()`/`NewXYZ()` 可以出现在类型定义之后、接收器方法之前。

由于函数按接收器分组，纯工具函数应放到文件末尾附近。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Reduce Nesting

尽量通过先处理错误/特殊情况并提前返回或 continue 来减少嵌套层级，避免多层嵌套代码。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Unnecessary Else

如果 if 的两个分支都给同一个变量赋值，可以用一个 if 替代。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Top-level Variable Declarations

在顶层声明中使用标准的 `var` 关键字。除非表达式类型与期望类型不同，否则不要显式指定类型。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

当表达式的类型与目标类型不一致时，需显式指定类型。

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

### Prefix Unexported Globals with _

对未导出的顶层 `var` 和 `const` 使用 `_` 前缀，便于识别它们是全局符号。

理由：顶层变量和常量具有包级作用域，泛化名称容易在其它文件中误用。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // We will not see a compile error if the first line of
  // Bar() is deleted.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

**例外**：未导出的 error 值可以使用 `err` 前缀而不是下划线。参见 [Error Naming](#error-naming)。

### Embedding in Structs

嵌入类型应放在 struct 字段列表的顶部，并与常规字段之间留一行空行。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

嵌入应带来明确收益，例如以语义上合理的方式扩展/增强功能，并且对用户侧没有负面影响（另见：[Avoid Embedding Types in Public Structs](#avoid-embedding-types-in-public-structs)）。

例外：mutex 不应被嵌入，即便是未导出类型。参见 [Zero-value Mutexes are Valid](#zero-value-mutexes-are-valid)。

**不应该** 为了以下目的嵌入：

- 纯粹为了美观或便捷。
- 让外层类型更难构造或使用。
- 影响外层类型的零值语义。如果外层类型本来有可用零值，嵌入后也应保持。
- 通过嵌入让外层类型暴露不相关的方法或字段。
- 暴露未导出类型。
- 影响外层类型的拷贝语义。
- 改变外层类型的 API 或类型语义。
- 嵌入内层类型的非规范形态。
- 暴露外层类型的实现细节。
- 让用户能够观察或控制类型内部细节。
- 通过包装改变内层函数的通用行为，且会让用户感到意外。

一句话：要有意识、谨慎地嵌入。一个简单的试金石是：“这些被导出的内层方法/字段是否都会被直接加到外层类型上？”如果答案是“部分”或“否”，就不要嵌入，用普通字段代替。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type A struct {
    // Bad: A.Lock() and A.Unlock() are
    //      now available, provide no
    //      functional benefit, and allow
    //      users to control details about
    //      the internals of A.
    sync.Mutex
}
```

</td><td>

```go
type countingWriteCloser struct {
    // Good: Write() is provided at this
    //       outer layer for a specific
    //       purpose, and delegates work
    //       to the inner type's Write().
    io.WriteCloser

    count int
}

func (w *countingWriteCloser) Write(bs []byte) (int, error) {
    w.count += len(bs)
    return w.WriteCloser.Write(bs)
}
```

</td></tr>
<tr><td>

```go
type Book struct {
    // Bad: pointer changes zero value usefulness
    io.ReadWriter

    // other fields
}

// later

var b Book
b.Read(...)  // panic: nil pointer
b.String()   // panic: nil pointer
b.Write(...) // panic: nil pointer
```

</td><td>

```go
type Book struct {
    // Good: has useful zero value
    bytes.Buffer

    // other fields
}

// later

var b Book
b.Read(...)  // ok
b.String()   // ok
b.Write(...) // ok
```

</td></tr>
<tr><td>

```go
type Client struct {
    sync.Mutex
    sync.WaitGroup
    bytes.Buffer
    url.URL
}
```

</td><td>

```go
type Client struct {
    mtx sync.Mutex
    wg  sync.WaitGroup
    buf bytes.Buffer
    url url.URL
}
```

</td></tr>
</tbody></table>

### Local Variable Declarations

如果变量要被显式赋值，优先使用短变量声明（`:=`）。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

但在一些场景中使用 `var` 的默认值更清晰，例如 [Declaring Empty Slices](https://go.dev/wiki/CodeReviewComments#declaring-empty-slices)。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### nil is a valid slice

`nil` 是合法的长度为 0 的 slice。因此：

- 不要显式返回长度为 0 的 slice，应返回 `nil`。

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- 判断 slice 是否为空应使用 `len(s) == 0`，不要判断 `nil`。

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- 零值（用 `var` 声明的 slice）可直接使用，无需 `make()`。

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)
  
  if add1 {
    nums = append(nums, 1)
  }
  
  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int
  
  if add1 {
    nums = append(nums, 1)
  }
  
  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

记住：虽然 `nil` slice 合法，但它与长度为 0 的已分配 slice 并不等价：一个为 nil，一个非 nil，在不同场景（如序列化）可能表现不同。

### Reduce Scope of Variables

尽量缩小变量与常量的作用域，但不要与 [Reduce Nesting](#reduce-nesting) 冲突。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
err := os.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := os.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

如果你需要在 if 外使用函数返回结果，就不要强行缩小作用域。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := os.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := os.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

常量除非在多个函数或文件中使用，或是包对外契约的一部分，否则不必是全局的。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
const (
  _defaultPort = 8080
  _defaultUser = "user"
)

func Bar() {
  fmt.Println("Default port", _defaultPort)
}
```

</td><td>

```go
func Bar() {
  const (
    defaultPort = 8080
    defaultUser = "user"
  )
  fmt.Println("Default port", defaultPort)
}
```

</td></tr>
</tbody></table>

### Avoid Naked Parameters

函数调用中使用裸参数会影响可读性。当参数含义不明显时，使用 C 风格注释（`/* ... */`）标注参数名。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

更好的做法：用自定义类型替代裸 `bool`，提高可读性与类型安全，也能在未来支持多于两种状态。

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady Status = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

### Use Raw String Literals to Avoid Escaping

Go 支持 [raw string literals](https://go.dev/ref/spec#raw_string_lit)，可以跨多行且包含引号。使用它们可避免手动转义带来的可读性问题。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Initializing Structs

#### Use Field Names to Initialize Structs

初始化 struct 时几乎总应写字段名。现在 [`go vet`](https://pkg.go.dev/cmd/vet)（静态分析）也会强制这一点。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

例外：在测试表中，如果字段数不超过 3 个，可以省略字段名。

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

#### Omit Zero Value Fields in Structs

按字段名初始化 struct 时，除非字段名本身提供重要语义，否则应省略零值字段，让 Go 自动设为零值。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
user := User{
  FirstName: "John",
  LastName: "Doe",
  MiddleName: "",
  Admin: false,
}
```

</td><td>

```go
user := User{
  FirstName: "John",
  LastName: "Doe",
}
```

</td></tr>
</tbody></table>

这样能减少阅读噪音，突出有意义的值。

当字段名能提供有价值的上下文时，应保留零值。例如 [Test Tables](#test-tables) 中的用例，即便是零值也能通过字段名表达含义。

```go
tests := []struct{
  give string
  want int
}{
  {give: "0", want: 0},
  // ...
}
```

#### Use `var` for Zero Value Structs

当 struct 声明中省略所有字段时，使用 `var` 形式声明。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
user := User{}
```

</td><td>

```go
var user User
```

</td></tr>
</tbody></table>

这能区分“零值 struct”与“有非零字段的 struct”，类似 [map 初始化](#initializing-maps) 的区分方式，也与我们偏好的 [声明空 slice](https://go.dev/wiki/CodeReviewComments#declaring-empty-slices) 方式一致。

#### Initializing Struct References

初始化 struct 引用时使用 `&T{}` 而不是 `new(T)`，以保持与 struct 初始化的一致性。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Initializing Maps

空 map 和需要程序化填充的 map，优先使用 `make(..)` 初始化。这样 map 初始化在视觉上更清晰，也便于后续添加容量提示。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

声明与初始化在视觉上很相似。

</td><td>

声明与初始化在视觉上更易区分。

</td></tr>
</tbody></table>

尽可能为 `make()` 初始化的 map 提供容量提示。详见 [Specifying Map Capacity Hints](#specifying-map-capacity-hints)。

另一方面，如果 map 保存的是固定集合，使用 map literal 初始化。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

基本原则：初始化时要放固定元素，就用 map literal；否则用 `make`（并在可能时提供容量提示）。

### Format Strings outside Printf

如果 `Printf` 风格函数的格式字符串不是写在字面量里，而是抽出来声明，请用 `const`。

这有助于 `go vet`（静态分析）对格式字符串做静态检查。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

### Naming Printf-style Functions

声明 `Printf` 风格函数时，确保 `go vet`（静态分析）能识别并检查格式字符串。

这意味着尽量使用预定义的 `Printf` 风格函数名，`go vet` 默认会检查这些名称。详见 [Printf family](https://pkg.go.dev/cmd/vet#hdr-Printf_family)。

如果不能使用预定义名称，你自定义的名称必须以 f 结尾：`Wrapf` 而不是 `Wrap`。`go vet` 可以被配置去检查指定的 `Printf` 风格名称，但它们必须以 f 结尾。

```shell
go vet -printfuncs=wrapf,statusf
```

另见 [go vet: Printf family check](https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/)。

## Patterns

### Test Tables

基于 [subtests](https://go.dev/blog/subtests) 的表驱动测试是一个很实用的模式，可以避免重复的测试代码。

当被测系统需要在 *多种条件* 下测试，且输入/输出只有部分变化时，使用表驱动测试可以减少冗余并提升可读性。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

测试表让错误信息更有上下文、减少重复逻辑，也更便于新增用例。

我们约定 struct slice 命名为 `tests`，每个测试用例命名为 `tt`。同时建议用 `give` 与 `want` 前缀明确每个用例的输入输出。

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

#### Avoid Unnecessary Complexity in Table Tests

如果子测试里包含条件断言或分支逻辑，表驱动测试会变得难读难维护。**不要** 在需要复杂或条件逻辑的子测试中使用表驱动（也就是 `for` 循环内存在复杂逻辑）。

大型、复杂的表驱动测试会损害可读性和可维护性，读者难以定位失败原因。

这类测试应拆成多个表或多个独立的 `Test...` 函数。

一些建议目标：

* 聚焦最小行为单元
* 降低“测试深度”，避免条件断言（见下）
* 确保表中字段在所有测试中都被使用
* 确保所有测试逻辑对所有用例都执行

这里的“测试深度”指“一个测试中依赖前置断言成立的连续断言数量”（类似圈复杂度）。深度越浅，断言之间的依赖越少，尤其是默认情况下更少出现条件断言。

具体来说，表驱动测试会变得难读的常见原因包括：使用多条分支路径（如 `shouldError`、`expectCall` 等）、用大量 `if` 做特定 mock 预期（如 `shouldCallFoo`）、或在表中放函数（如 `setupMocks func(*FooMock)`）。

但当被测行为仅随输入变化而变化时，把类似用例放在同一张表中更容易展示差异，也比拆成多个测试更便于比较。

如果测试主体短且直观，可以接受用一个分支字段（如 `shouldErr`）区分成功/失败用例。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func TestComplicatedTable(t *testing.T) {
  tests := []struct {
    give          string
    want          string
    wantErr       error
    shouldCallX   bool
    shouldCallY   bool
    giveXResponse string
    giveXErr      error
    giveYResponse string
    giveYErr      error
  }{
    // ...
  }

  for _, tt := range tests {
    t.Run(tt.give, func(t *testing.T) {
      // setup mocks
      ctrl := gomock.NewController(t)
      xMock := xmock.NewMockX(ctrl)
      if tt.shouldCallX {
        xMock.EXPECT().Call().Return(
          tt.giveXResponse, tt.giveXErr,
        )
      }
      yMock := ymock.NewMockY(ctrl)
      if tt.shouldCallY {
        yMock.EXPECT().Call().Return(
          tt.giveYResponse, tt.giveYErr,
        )
      }

      got, err := DoComplexThing(tt.give, xMock, yMock)

      // verify results
      if tt.wantErr != nil {
        require.EqualError(t, err, tt.wantErr)
        return
      }
      require.NoError(t, err)
      assert.Equal(t, want, got)
    })
  }
}
```

</td><td>

```go
func TestShouldCallX(t *testing.T) {
  // setup mocks
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)
  xMock.EXPECT().Call().Return("XResponse", nil)

  yMock := ymock.NewMockY(ctrl)

  got, err := DoComplexThing("inputX", xMock, yMock)

  require.NoError(t, err)
  assert.Equal(t, "want", got)
}

func TestShouldCallYAndFail(t *testing.T) {
  // setup mocks
  ctrl := gomock.NewController(t)
  xMock := xmock.NewMockX(ctrl)

  yMock := ymock.NewMockY(ctrl)
  yMock.EXPECT().Call().Return("YResponse", nil)

  _, err := DoComplexThing("inputY", xMock, yMock)
  assert.EqualError(t, err, "Y failed")
}
```
</td></tr>
</tbody></table>

这样的复杂度会让测试更难改、更难理解、更难验证正确性。

虽然没有硬性规定，但在选择表驱动测试还是拆分多测时，始终把可读性和可维护性放在首位。

#### Parallel Tests

并行测试（以及某些特殊循环，例如在循环体中启动 goroutine 或捕获引用）必须显式在循环作用域内绑定循环变量，保证其值符合预期。

```go
tests := []struct{
  give string
  // ...
}{
  // ...
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    t.Parallel()
    // ...
  })
}
```

上面的例子中，因为使用了 `t.Parallel()`，必须在循环迭代内声明一个作用域内的 `tt` 变量。否则多数（或全部）测试会拿到意外的 `tt` 值，或者 `tt` 会在运行时变化。

<!-- TODO: Explain how to use _test packages. -->

### Functional Options

函数式选项模式：定义一个不透明的 `Option` 类型，在内部 struct 上记录信息。调用方传入可变参数 options，在内部结构上统一应用这些配置。

当构造函数或其它公开 API 的可选参数可能扩展时，使用该模式，尤其是函数已有 3 个或更多参数时。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Open(
  addr string,
  cache bool,
  logger *zap.Logger
) (*Connection, error) {
  // ...
}
```

</td><td>

```go
// package db

type Option interface {
  // ...
}

func WithCache(c bool) Option {
  // ...
}

func WithLogger(log *zap.Logger) Option {
  // ...
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  // ...
}
```

</td></tr>
<tr><td>

cache 与 logger 参数总得传，即使用户想用默认值。

```go
db.Open(addr, db.DefaultCache, zap.NewNop())
db.Open(addr, db.DefaultCache, log)
db.Open(addr, false /* cache */, zap.NewNop())
db.Open(addr, false /* cache */, log)
```

</td><td>

只有需要时才传 options。

```go
db.Open(addr)
db.Open(addr, db.WithLogger(log))
db.Open(addr, db.WithCache(false))
db.Open(
  addr,
  db.WithCache(false),
  db.WithLogger(log),
)
```

</td></tr>
</tbody></table>

我们建议的实现方式是：`Option` interface 持有一个未导出方法，在未导出的 `options` struct 上记录选项。

```go
type options struct {
  cache  bool
  logger *zap.Logger
}

type Option interface {
  apply(*options)
}

type cacheOption bool

func (c cacheOption) apply(opts *options) {
  opts.cache = bool(c)
}

func WithCache(c bool) Option {
  return cacheOption(c)
}

type loggerOption struct {
  Log *zap.Logger
}

func (l loggerOption) apply(opts *options) {
  opts.logger = l.Log
}

func WithLogger(log *zap.Logger) Option {
  return loggerOption{Log: log}
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    cache:  defaultCache,
    logger: zap.NewNop(),
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}
```

也可以用闭包实现该模式，但我们认为上面的模式对作者更灵活、对使用者更易调试与测试。比如：它允许在测试与 mock 中比较 options，而闭包做不到；此外 options 还可以实现其它 interface（如 `fmt.Stringer`），便于输出用户可读的选项信息。

另见：

- [Self-referential functions and the design of options](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html)
- [Functional options for friendly APIs](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis)

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->

## Linting

与其纠结“官方推荐哪组 linter”，更重要的是在代码库内保持一致的 lint 标准。

我们建议至少使用以下 linter，因为它们能覆盖最常见问题，并在不过度苛刻的前提下把质量门槛拉高：

- [errcheck](https://github.com/kisielk/errcheck) 用于确保 error 被处理
- [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) 用于格式化代码并管理 import
- [revive](https://github.com/mgechev/revive) 用于指出常见风格问题
- [govet](https://pkg.go.dev/cmd/vet) 用于分析常见错误
- [staticcheck](https://staticcheck.dev) 用于进行多类静态分析

  > **Note**: [revive](https://github.com/mgechev/revive) is the modern, faster successor to the now-deprecated [golint](https://github.com/golang/lint).

### Lint Runners

我们推荐 [golangci-lint](https://github.com/golangci/golangci-lint) 作为 Go 代码的首选 lint runner，主要因为它在大型代码库中的性能，以及可同时配置并运行多种权威 linter。本仓库提供一个 [.golangci.yml](https://github.com/uber-go/guide/blob/master/.golangci.yml) 示例配置，包含推荐 linter 与设置。

golangci-lint 提供了 [多种 linter](https://golangci-lint.run/usage/linters/) 供选择。上述 linter 作为基础组合，我们也鼓励团队根据项目需要添加其它 linter。

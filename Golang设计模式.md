# Golang设计模式

Go语言惯用的设计和应用模式的精选集合。

## 创建模式

### 抽象工厂模式

提供一个用于创建一系列相关对象的接口

### 构建器模式

构建器模式将复杂对象的构造及其表示分开，以便相同的构造过程可以创建不同的表示。
在Go中，通常使用配置结构来实现相同的行为，但是将结构体传递给构建器方法会使用样板`if cfg.Field != nil {...}`检查填充代码。
* *Implementation*

```go
package car

type Speed float64

const (
    MPH Speed = 1
    KPH       = 1.60934
)

type Color string

const (
    BlueColor  Color = "blue"
    GreenColor       = "green"
    RedColor         = "red"
)

type Wheels string

const (
    SportsWheels Wheels = "sports"
    SteelWheels         = "steel"
)

type Builder interface {
    Color(Color) Builder
    Wheels(Wheels) Builder
    TopSpeed(Speed) Builder
    Build() Interface
}

type Interface interface {
    Drive() error
    Stop() error
}
```

* *Usage*
```go
assembly := car.NewBuilder().Paint(car.RedColor)

familyCar := assembly.Wheels(car.SportsWheels).TopSpeed(50 * car.MPH).Build()
familyCar.Drive()

sportsCar := assembly.Wheels(car.SteelWheels).TopSpeed(150 * car.MPH).Build()
sportsCar.Drive()
```

### 工厂方法模式

工厂方法创建设计模式允许创建对象，而无需指定将要创建的对象的确切类型。

* *Implementation*
示例实现显示了如何为数据存储提供不同的后端，例如内存、磁盘存储。

```go
package data

import "io"

type Store interface {
    Open(string) (io.ReadWriteCloser, error)
}

// OR

package data

type StorageType int

const (
    DiskStorage StorageType = 1 << iota
    TempStorage
    MemoryStorage
)

func NewStore(t StorageType) Store {
    switch t {
    case MemoryStorage:
        return newMemoryStorage( /*...*/ )
    case DiskStorage:
        return newDiskStorage( /*...*/ )
    default:
        return newTempStorage( /*...*/ )
    }
}
```

* *Usage*
使用工厂方法，用户可以指定他们想要的存储类型。

```go
s, _ := data.NewStore(data.MemoryStorage)
f, _ := s.Open("file")

n, _ := f.Write([]byte("data"))
defer f.Close()
```

### 对象池模式

实例化并维护一组相同类型的对象实例，对象池创建设计模式用于根据需求期望准备和保留多个实例。

* *Implementation*
```go
package pool

type Pool chan *Object

func New(total int) *Pool {
    p := make(Pool, total)

    for i := 0; i < total; i++ {
        p <- new(Object)
    }

    return &p
}
```

* *Usage*
下面给出了一个对象池上的简单生命周期示例。

```go
p := pool.New(2)

select {
case obj := <-p:
    obj.Do( /*...*/ )

    p <- obj
default:
    // No more objects left — retry later or fail
    return
}
```

* *经验法则*
	* 在对象初始化比对象维护更昂贵的情况下，对象池模式很有用。
	* 如果需求激增而不是稳定需求，则维护开销可能会超过对象池的优势。
	* 由于预先初始化对象，它对性能有积极影响。

### 单例模式Singleton

Singleton创建设计模式将类型的实例化限制为单个对象。

* *Implementation*
```go
package singleton

type singleton map[string]string

var (
    once sync.Once

    instance singleton
)

func New() singleton {
    once.Do(func() {
        instance = make(singleton)
    })

    return instance
}
```

* *Usage*

```go
s := singleton.New()

s["this"] = "that"

s2 := singleton.New()

fmt.Println("This is ", s2["this"])
// This is that
```

* *经验法则*

单例模式表示全局状态，大多数情况下会降低可测试性。

## 结构模式

### 装饰模式

装饰器结构模式允许动态扩展现有对象的功能而不改变其内部状态。
装饰器提供了一种灵活的方法来扩展对象的功能。

* *Implementation*

`LogDecorate` 装饰一个签名为 `func(int) int` 的函数来处理整数，并添加输入/输出日志记录功能。
```golang
type Object func(int) int

func LogDecorate(fn Object) Object {
    return func(n int) int {
        log.Println("Starting the execution with the integer", n)

        result := fn(n)

        log.Println("Execution is completed with the result", result)

        return result
    }
}
```

* *Usage*

```golang
func Double(n int) int {
    return n * 2
}

f := LogDecorate(Double)

f(5)
// Starting execution with the integer 5
// Execution is completed with the result 10
```

* *经验法则*

    * 与适配器模式不同，要装饰的对象是通过注入(injection)获得的。
    * 装饰者不应该改变对象的接口。

### 代理模式

代理模式提供了控制访问到另一个对象，拦截所有调用的对象。

* *Implementation*

代理可以为任何东西实现接口：网络连接，内存中的大对象，文件以及其他昂贵或无法复制的资源。
简单的实例：

```golang
    // To use proxy and to object they must implement same methods
    type IObject interface {
        ObjDo(action string)
    }

    // Object represents real objects which proxy will delegate data
    type Object struct {
        action string
    }

    // ObjDo implements IObject interface and handel's all logic
    func (obj *Object) ObjDo(action string) {
        // Action behavior
        fmt.Printf("I can, %s", action)
    }

    // ProxyObject represents proxy object with intercepts actions
    type ProxyObject struct {
        object *Object
    }

    // ObjDo are implemented IObject and intercept action before send in real Object
    func (p *ProxyObject) ObjDo(action string) {
        if p.object == nil {
            p.object = new(Object)
        }
        if action == "Run" {
            p.object.ObjDo(action) // Prints: I can, Run
        }
    }
```

* *Usage*

More complex usage of proxy as example: User creates "Terminal" authorizes and PROXY send execution command to real Terminal object See [proxy/main.go](https://hxangel.gitbooks.io/go-patterns/content/structural/proxy/main.go) or [view in the Playground](https://play.golang.org/p/mnjKCMaOVE).


## 行为模式

### 观察者模式

在观察者模式允许类型实例“发布”事件到其他类型实例（观察者），观察者希望当特定事件发生时进行更新操作。

* *Implementation*

在长期运行的应用程序中 - 例如webservers-instances可以保留一组观察者，这些观察者将接收触发事件的通知。
实现方式各不相同，但接口可用于制作标准观察者和通知者：

```golang
type (
    // Event defines an indication of a point-in-time occurrence.
    Event struct {
        // Data in this case is a simple int, but the actual
        // implementation would depend on the application.
        Data int64
    }

    // Observer defines a standard interface for instances that wish to list for
    // the occurrence of a specific event.
    Observer interface {
        // OnNotify allows an event to be "published" to interface implementations.
        // In the "real world", error handling would likely be implemented.
        OnNotify(Event)
    }

    // Notifier is the instance being observed. Publisher is perhaps another decent
    // name, but naming things is hard.
    Notifier interface {
        // Register allows an instance to register itself to listen/observe
        // events.
        Register(Observer)
        // Deregister allows an instance to remove itself from the collection
        // of observers/listeners.
        Deregister(Observer)
        // Notify publishes new events to listeners. The method is not
        // absolutely necessary, as each implementation could define this itself
        // without losing functionality.
        Notify(Event)
    }
)
```

* *Usage*

For usage, see [observer/main.go](https://hxangel.gitbooks.io/go-patterns/behavioral/observer/main.go) or [view in the Playground](https://play.golang.org/p/cr8jEmDmw0).

### 策略模式

策略行为设计模式允许在运行时选择算法的行为。
它定义算法，封装它们，并互换使用它们。

* *Implementation*

实现对整数进行操作的可互换运算符对象。

```golang
type Operator interface {
    Apply(int, int) int
}

type Operation struct {
    Operator Operator
}

func (o *Operation) Operate(leftValue, rightValue int) int {
    return o.Operator.Apply(leftValue, rightValue)
}
```

* *Usage*
    * Addition Operator

```golang
type Addition struct{}

func (Addition) Apply(lval, rval int) int {
    return lval + rval
}


add := Operation{Addition{}}
add.Operate(3, 5) // 8
```

    * Multiplication Operator

```golang 
type Multiplication struct{}

func (Multiplication) Apply(lval, rval int) int {
    return lval * rval
}


mult := Operation{Multiplication{}}

mult.Operate(3, 5) // 15
```

* *经验法则*

    * 除其粒度外，策略模式与模板模式类似。
    * 策略模式允许您更改对象的内容，而装饰器模式改变皮肤。


## 同步模式

### 信号量模式

信号量是同步模式/原语，它在有限数量的资源上强加互斥。

* *Implementation*

```golang
package semaphore

var (
    ErrNoTickets      = errors.New("semaphore: could not aquire semaphore")
    ErrIllegalRelease = errors.New("semaphore: can't release the semaphore without acquiring it first")
)

// Interface contains the behavior of a semaphore that can be acquired and/or released.
type Interface interface {
    Acquire() error
    Release() error
}

type implementation struct {
    sem     chan struct{}
    timeout time.Duration
}

func (s *implementation) Acquire() error {
    select {
    case s.sem <- struct{}{}:
        return nil
    case <-time.After(s.timeout):
        return ErrNoTickets
    }
}

func (s *implementation) Release() error {
    select {
    case _ = <-s.sem:
        return nil
    case <-time.After(s.timeout):
        return ErrIllegalRelease
    }

    return nil
}

func New(tickets int, timeout time.Duration) Interface {
    return &implementation{
        sem:     make(chan struct{}, tickets),
        timeout: timeout,
    }
}
```

* *Usage*

    * 超时的信号量
```golang
tickets, timeout := 1, 3*time.Second
s := semaphore.New(tickets, timeout)

if err := s.Acquire(); err != nil {
    panic(err)
}

// Do important work

if err := s.Release(); err != nil {
    panic(err)
}
```
    * 没有超时的信号量（非阻塞）
```golang
tickets, timeout := 0, 0
s := semaphore.New(tickets, timeout)

if err := s.Acquire(); err != nil {
    if err != semaphore.ErrNoTickets {
        panic(err)
    }

    // No tickets left, can't work :(
    os.Exit(1)
}
```


## 并发模式

### 生成器模式

Generators yields a sequence of values one at a time.

* *Implementation*

```golang
func Count(start int, end int) chan int {
    ch := make(chan int)

    go func(ch chan int) {
        for i := start; i <= end ; i++ {
            // Blocks on the operation
            ch <- i
        }

        close(ch)
    }(ch)

    return ch
}
```

* *Usage*

```golang
fmt.Println("No bottles of beer on the wall")

for i := range Count(1, 99) {
    fmt.Println("Pass it around, put one up,", i, "bottles of beer on the wall")
    // Pass it around, put one up, 1 bottles of beer on the wall
    // Pass it around, put one up, 2 bottles of beer on the wall
    // ...
    // Pass it around, put one up, 99 bottles of beer on the wall
}

fmt.Println(100, "bottles of beer on the wall")
```

### 并行模式

并行性允许多个“作业”或任务同时和异步地运行。

* *Implementation and Example*

可以在[parallelism.go](https://hxangel.gitbooks.io/go-patterns/content/concurrency/parallelism.go)中找到实现和使用的示例。

## 消息模式

### Fan-In消息传递模式

Fan-In是一种消息传递模式，用于为工作者（客户端：源，服务器：目标）之间的工作创建漏斗。
我们可以使用Go channel 模拟 Fan-In。

```golang
// Merge different channels in one channel
func Merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup

    out := make(chan int)

    // Start an send goroutine for each input channel in cs. send
    // copies values from c to out until c is closed, then calls wg.Done.
    send := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }

    wg.Add(len(cs))
    for _, c := range cs {
        go send(c)
    }

    // Start a goroutine to close out once all the send goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

该`Merge`函数通过为每个入站通道启动goroutine将一个通道列表转换为单个通道，该通道将值复制到唯一的出站通道。
一旦启动了所有输出goroutine，`Merge`就会启动goroutine来关闭主通道。


### Fan-Out消息传递模式

Fan-Out是一种消息传递模式，用于在工作者之间分配工作（生产者：来源，消费者：目的地）。
我们可以使用Go channel 模拟 Fan-Out。

```golang
// Split a channel into n channels that receive messages in a round-robin fashion.
func Split(ch <-chan int, n int) []<-chan int {
    cs := make([]chan int)
    for i := 0; i < n; i++ {
        cs = append(cs, make(chan int))
    }

    // Distributes the work in a round robin fashion among the stated number
    // of channels until the main channel has been closed. In that case, close
    // all channels and return.
    distributeToChannels := func(ch <-chan int, cs []chan<- int) {
        // Close every channel when the execution ends.
        defer func(cs []chan<- int) {
            for _, c := range cs {
                close(c)
            }
        }(cs)

        for {
            for _, c := range cs {
                select {
                case val, ok := <-ch:
                    if !ok {
                        return
                    }

                    c <- val
                }
            }
        }
    }

    go distributeToChannels(ch, cs)

    return cs
}
```

`Split`函数通过使用goroutine以循环方式将接收到的值复制到列表中的channel，将单个通道转换为通道列表。

### 发布和订阅消息传递模式

Publish-Subscribe是一种消息传递模式，用于在不同组件之间传递消息，而这些组件不了解彼此的身份。

它类似于Observer行为设计模式。Observer和Publish-Subscribe的基本设计原则是解耦那些对来自‘线人’（观察者或发布者）的`Event Messages`有兴趣并获取通知的方式。这意味着您不必将消息编程以便直接发送给特定的接收器。

为实现此目的，称为“消息代理”或“事件总线”的中介接收已发布的消息，然后将它们路由到订阅者。

有三个组件消息，主题，用户。

```golang
type Message struct {
    // Contents
}


type Subscription struct {
    ch chan<- Message

    Inbox chan Message
}

func (s *Subscription) Publish(msg Message) error {
    if _, ok := <-s.ch; !ok {
        return errors.New("Topic has been closed")
    }

    s.ch <- msg

    return nil
}
```

```golang
type Topic struct {
    Subscribers    []Session
    MessageHistory []Message
}

func (t *Topic) Subscribe(uid uint64) (Subscription, error) {
    // Get session and create one if it's the first

    // Add session to the Topic & MessageHistory

    // Create a subscription
}

func (t *Topic) Unsubscribe(Subscription) error {
    // Implementation
}

func (t *Topic) Delete() error {
    // Implementation
}
```

```golang
type User struct {
    ID uint64
    Name string
}

type Session struct {
    User User
    Timestamp time.Time
}
```

* *改进*

通过利用无堆叠的goroutine，可以以并行方式发布事件。

通过使用缓冲的收件箱处理订阅者意外掉队的情况并在收件箱已满后停止发送事件，可以提高性能。

## 稳定模式

### 断路器模式

类似于电气保险丝，当连接到电网的电路开始吸收大量电力导致电线加热和燃烧时，可以防止火灾，断路器设计模式是一种关闭电路、软件开发时的请求/响应关系或服务的故障优先机制，以防止更大的故障。

*注意：* 在本文档中，“电路”和“服务”一词同义使用。

* *Implementation*

下面是一个非常简单的断路器的实现，以说明断路器设计模式的目的。

    * Operation Counter
    `circuit.Counter` 是一个简单的计数器，记录电路的成功和失败状态以及时间戳，并计算连续的故障数。
```golang
package circuit

import (
    "time"
)

type State int

const (
    UnknownState State = iota
    FailureState
    SuccessState
)

type Counter interface {
    Count(State)
    ConsecutiveFailures() uint32
    LastActivity() time.Time
    Reset()
}
```

    * Circuit Breaker断路器

    电路使用`circuit.Breaker`保持内部操作计数器并闭合电路。如果电路连续失败超过指定阈值，则返回快速错误。过一会儿，它会重试请求并记录下来。
    *注意：* 此处使用上下文类型来跨API边界和进程之间传送截止日期，取消信号和其他请求范围的值。

```golang
package circuit

import (
    "context"
    "time"
)

type Circuit func(context.Context) error

func Breaker(c Circuit, failureThreshold uint32) Circuit {
    cnt := NewCounter()

    return func(ctx context) error {
        if cnt.ConsecutiveFailures() >= failureThreshold {
            canRetry := func(cnt Counter) {
                backoffLevel := Cnt.ConsecutiveFailures() - failureThreshold

                // Calculates when should the circuit breaker resume propagating requests
                // to the service
                shouldRetryAt := cnt.LastActivity().Add(time.Seconds * 2 << backoffLevel)

                return time.Now().After(shouldRetryAt)
            }

            if !canRetry(cnt) {
                // Fails fast instead of propagating requests to the circuit since
                // not enough time has passed since the last failure to retry
                return ErrServiceUnavailable
            }
        }

        // Unless the failure threshold is exceeded the wrapped service mimics the
        // old behavior and the difference in behavior is seen after consecutive failures
        if err := c(ctx); err != nil {
            cnt.Count(FailureState)
            return err
        }

        cnt.Count(SuccessState)
        return nil
    }
}
```

* *相关应用*

[sony/gobreaker](https://github.com/sony/gobreaker)是针对实际用例的经过充分测试且直观的断路器实现。

## 剖析模式

### 时间功能

在优化代码时，有时需要快速而肮脏的时间测量，而不是利用分析器工具/框架来验证假设。

可以通过利用`time`包和`defer`语句来执行时间测量。

* *Implementation*

```golang
package profile

import (
    "time"
    "log"
)

func Duration(invocation time.Time, name string) {
    elapsed := time.Since(invocation)

    log.Printf("%s lasted %s", name, elapsed)
}
```

* *Usage*

```golang
func BigIntFactorial(x big.Int) *big.Int {
    // Arguments to a defer statement is immediately evaluated and stored.
    // The deferred function receives the pre-evaluated values when its invoked.
    defer profile.Duration(time.Now(), "IntFactorial")

    y := big.NewInt(1)
    for one := big.NewInt(1); x.Sign() > 0; x.Sub(x, one) {
        y.Mul(y, x)
    }

    return x.Set(y)
}
```

## 谚语模式

### 函数选项

函数选项是在Go中实现干净/清晰的API的方法。作为函数实现的选项设置该选项的状态。

* *Implementation*

    * Options
```golang
package file

type Options struct {
    UID         int
    GID         int
    Flags       int
    Contents    string
    Permissions os.FileMode
}

type Option func(*Options)

func UID(userID int) Option {
    return func(args *Options) {
        args.UID = userID
    }
}

func GID(groupID int) Option {
    return func(args *Options) {
        args.GID = groupID
    }
}

func Contents(c string) Option {
    return func(args *Options) {
        args.Contents = c
    }
}

func Permissions(perms os.FileMode) Option {
    return func(args *Options) {
        args.Permissions = perms
    }
}
```

    * Constructor

```golang
package file

func New(filepath string, setters ...Option) error {
    // Default Options
    args := &Options{
        UID:         os.Getuid(),
        GID:         os.Getgid(),
        Contents:    "",
        Permissions: 0666,
        Flags:       os.O_CREATE | os.O_EXCL | os.O_WRONLY,
    }

    for _, setter := range setters {
        setter(args)
    }

    f, err := os.OpenFile(filepath, args.Flags, args.Permissions)
    if err != nil {
        return err
    } else {
        defer f.Close()
    }

    if _, err := f.WriteString(args.Contents); err != nil {
        return err
    }

    return f.Chown(args.UID, args.GID)
}
```

* *Usage*

```golang
emptyFile, err := file.New("/tmp/empty.txt")
if err != nil {
    panic(err)
}

fillerFile, err := file.New("/tmp/file.txt", file.UID(1000), file.Contents("Lorem Ipsum Dolor Amet"))
if err != nil {
    panic(err)
}
```


































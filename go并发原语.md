# Go 并发原语



Go是一门支持并发编程的编程语言，它提供了丰富的并发原语，使得编写并发程序变得相对容易。以下是一些Go中常用的并发原语：

### Goroutine（协程）

- Goroutine 是Go中的轻量级线程，允许在程序中同时执行多个函数或方法。它们由Go运行时管理，相对于传统的线程，它们的创建和销毁成本较低。

- 通过关键字 `go` 来启动一个新的Goroutine。例如：`go someFunction()`

  



### Channels（通道）

- 通道是Golang中用于在Goroutines之间进行通信和同步的机制。通道允许一个Goroutine发送数据到通道，同时另一个Goroutine可以从通道接收数据。
- 通过使用 `make` 函数来创建通道：`ch := make(chan int)`



### Select 语句：

- `select` 语句用于从多个通道中选择一个非阻塞的接收操作。它用于处理多个并发任务的结果，可以防止Goroutines因等待某个通道而被阻塞。
- 例如：

```go
goCopy codeselect {
    case result1 := <-ch1:
        // 处理来自ch1的数据
    case result2 := <-ch2:
        // 处理来自ch2的数据
    default:
        // 如果没有通道准备好，执行默认操作
}
```



### Mutex（互斥锁）：

- Golang提供了互斥锁（Mutex）来实现对共享资源的互斥访问。通过在临界区代码块前后使用锁，可以确保在同一时刻只有一个Goroutine可以访问共享资源。
- 例如：

```go
goCopy codevar mu sync.Mutex
mu.Lock()
// 访问共享资源
mu.Unlock()
```



### WaitGroup（等待组）：

- `sync.WaitGroup` 用于等待一组Goroutines完成执行。它允许主Goroutine 等待所有其他Goroutines 完成工作。
- 例如：

```go
goCopy codevar wg sync.WaitGroup
wg.Add(1) // 增加等待计数
go func() {
    defer wg.Done() // 减少等待计数
    // 执行一些工作
}()
wg.Wait() // 等待所有Goroutines 完成
```

这些是Golang中常用的并发原语，它们使得编写并发程序变得更加简单和可控。通过有效地使用这些原语，可以实现高效的并发应用程序。请根据您的需求选择适当的原语来构建并发代码。



### Cond (条件信号)

在Go语言中，`sync.Cond` 是标准库 `sync` 包提供的条件变量（Condition Variable）类型，用于在多个Goroutines之间实现更高级的同步和通信。条件变量通常与互斥锁（Mutex）一起使用，以实现等待某些条件满足后才继续执行的并发控制。

`sync.Cond` 类型具有以下常用方法：

1. `func NewCond(l Locker) *Cond`：
   - 创建一个新的条件变量，其中 `l` 是任何实现了 `sync.Locker` 接口的锁（通常是 `sync.Mutex` 或 `sync.RWMutex`）。这个锁将与条件变量一起使用，以实现安全的等待和通知操作。
2. `func (c *Cond) Wait()`：
   - 在条件变量上等待通知，会将调用Goroutine阻塞，直到另一个Goroutine 调用 `Signal` 或 `Broadcast` 方法通知条件变量的等待者可以继续执行。在等待期间，条件变量会释放与其关联的锁，以便其他Goroutines 可以获得该锁。
3. `func (c *Cond) Signal()`：
   - 用于通知等待在条件变量上的一个Goroutine可以继续执行。如果有多个Goroutines在等待，只有其中一个会被唤醒，但是不能确定唤醒的是哪一个。通常，`Signal` 被用于通知单个等待者。
4. `func (c *Cond) Broadcast()`：
   - 用于通知等待在条件变量上的所有Goroutines可以继续执行。它会唤醒所有等待者。

下面是简单示例，演示如何使用 `sync.Cond` 来实现条件变量：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var (
		mutex sync.Mutex
		cond  *sync.Cond = sync.NewCond(&mutex)
	)

	// 启动一个等待Goroutine
	go func() {
		fmt.Println("Waiting for signal...")
		mutex.Lock()
		defer mutex.Unlock()
		cond.Wait() // 等待条件变量的通知
		fmt.Println("Received signal!")
	}()

	// 模拟一些工作
	time.Sleep(2 * time.Second)

	// 发送信号给等待的Goroutine
	mutex.Lock()
	defer mutex.Unlock()
	fmt.Println("Sending signal...")
	cond.Signal() // 发送信号给等待的Goroutine

	// 主Goroutine等待一段时间
	time.Sleep(2 * time.Second)
}
```

 ```go
 package main
 
 import (
 	"fmt"
 	"sync"
 	"time"
 )
 
 func main() {
 	var (
 		mutex sync.Mutex
 		cond  *sync.Cond = sync.NewCond(&mutex)
 	)
 
 	// 启动多个等待Goroutines
 	for i := 0; i < 3; i++ {
 		go func(id int) {
 			fmt.Printf("Goroutine %d: Waiting for broadcast...\n", id)
 			mutex.Lock()
 			defer mutex.Unlock()
 			cond.Wait() // 等待条件变量的通知
 			fmt.Printf("Goroutine %d: Received broadcast!\n", id)
 		}(i)
 	}
 
 	// 模拟一些工作
 	time.Sleep(2 * time.Second)
 
 	// 发送广播信号给所有等待的Goroutines
 	mutex.Lock()
 	defer mutex.Unlock()
 	fmt.Println("Sending broadcast signal...")
 	cond.Broadcast() // 发送广播信号给所有等待的Goroutines
 
 	// 主Goroutine等待一段时间
 	time.Sleep(2 * time.Second)
 }
 
 ```



在这个示例中，创建了一个条件变量 `cond`，并启动了一个等待Goroutine，它会等待条件变量上的通知。然后，主Goroutine等待一段时间后发送了一个信号，通知等待Goroutine可以继续执行。

请注意，`sync.Cond` 通常与锁（通常是 `sync.Mutex` 或 `sync.RWMutex`）一起使用，以确保在等待和通知期间的并发安全性。这有助于协调多个Goroutines 之间的工作流程。



### sync.Once (单例)

`sync.Once` 是 Go 语言标准库 `sync` 包中的一个类型，用于确保某个函数只会被执行一次，无论有多少 Goroutines 调用它。这对于执行初始化操作或设置全局状态的场景非常有用。

`sync.Once` 类型有一个 `Do` 方法，该方法接受一个函数作为参数，该函数会在第一次调用 `Do` 方法时执行，而后续的调用将被忽略。

以下是 `sync.Once` 的基本用法示例：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once

	for i := 0; i < 3; i++ {
		// 使用 sync.Once 来确保函数只会执行一次
		once.Do(func() {
			fmt.Println("This will only run once.")
		})
		fmt.Println("This will run every time.")
	}
}
```

在这个示例中，`Do` 方法确保传递给它的函数只会执行一次，即使它被多个 Goroutines 调用。因此，无论循环运行多少次，`This will only run once.` 只会被打印一次。

`sync.Once` 在需要执行一次性初始化、单例模式或全局状态的场景中非常有用。它提供了一种线程安全的方式来保证初始化代码仅执行一次。



### atomic 

`atomic` 包是 Go 语言标准库中的一个包，它提供了原子操作，用于在多个 Goroutines 之间安全地读取和写入共享的变量。原子操作是一种特殊的操作，它在执行期间不会被中断，因此不会发生竞态条件（Race Condition）。这在多线程编程中非常重要，因为避免竞态条件可以确保程序的安全性。

`atomic` 包提供了一些常用的原子操作，这些操作通常用于处理整数类型（例如 `int32`、`int64`）和指针。以下是一些常用的 `atomic` 包函数和操作：

1. `func LoadInt32(addr *int32) (val int32)`:
   - 用于原子加载 `int32` 类型的变量的值。
2. `func LoadInt64(addr *int64) (val int64)`:
   - 用于原子加载 `int64` 类型的变量的值。
3. `func StoreInt32(addr *int32, val int32)`:
   - 用于原子存储 `int32` 类型的变量的值。
4. `func StoreInt64(addr *int64, val int64)`:
   - 用于原子存储 `int64` 类型的变量的值。
5. `func AddInt32(addr *int32, delta int32) (new int32)`:
   - 原子地将 `delta` 添加到 `int32` 变量的值中，并返回新的值。
6. `func AddInt64(addr *int64, delta int64) (new int64)`:
   - 原子地将 `delta` 添加到 `int64` 变量的值中，并返回新的值。
7. `func SwapInt32(addr *int32, new int32) (old int32)`:
   - 原子地将 `int32` 变量的值设置为 `new` 并返回旧值。
8. `func SwapInt64(addr *int64, new int64) (old int64)`:
   - 原子地将 `int64` 变量的值设置为 `new` 并返回旧值。
9. `func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)`:
   - 原子地比较 `int32` 变量的值和 `old`，如果相等则将其设置为 `new` 并返回 `true`，否则返回 `false`。
10. `func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)`:
    - 原子地比较 `int64` 变量的值和 `old`，如果相等则将其设置为 `new` 并返回 `true`，否则返回 `false`。

这些函数提供了一种安全且高效的方式来执行原子操作，而不需要显式地使用锁（如 `sync.Mutex`）来保护共享的变量。原子操作适用于需要频繁访问和修改共享状态的并发应用程序。

以下是一些使用 `atomic` 包的示例应用：

1. **计数器**：

   假设有多个 Goroutines 需要增加一个共享的计数器的值。在没有原子操作的情况下，可能需要使用互斥锁来保护计数器。但是，使用 `atomic` 包可以更加高效地完成这项任务。

   ```go
   package main
   
   import (
       "fmt"
       "sync/atomic"
       "time"
   )
   
   func main() {
       var counter int32
   
       for i := 0; i < 10; i++ {
           go func() {
               atomic.AddInt32(&counter, 1)
           }()
       }
   
       time.Sleep(time.Second)
   
       fmt.Println("Counter:", atomic.LoadInt32(&counter))
   }
   ```

2. **标记位**：

   使用原子操作来设置和检查标记位非常常见。例如，可以使用 `atomic` 包来实现一种可停止的 Goroutine 控制机制。

   ```go
   package main
   
   import (
       "fmt"
       "sync/atomic"
       "time"
   )
   
   func main() {
       var done int32
   
       go func() {
           for {
               if atomic.LoadInt32(&done) == 1 {
                   fmt.Println("Goroutine: Exiting...")
                   break
               }
               // 执行一些工作
           }
       }()
   
       // 模拟一些工作
       time.Sleep(2 * time.Second)
   
       // 停止 Goroutine
       atomic.StoreInt32(&done, 1)
       fmt.Println("Main: Goroutine should stop soon.")
       time.Sleep(time.Second)
   }
   ```

   3.单例模式

单例模式使用 `CompareAndSwap` 操作来确保只有一个实例被创建的确实是一种无锁方式，因为它依赖于原子操作，而不需要显式的锁来保护共享资源。这是因为 `CompareAndSwap` 操作本身就是原子的，它在一个操作中比较变量的当前值和预期值，如果它们匹配，则将新值设置到变量中，所有这些步骤在一个不可中断的原子操作中完成。

以下是一个使用 `CompareAndSwap` 来实现无锁单例模式的示例：

```go
package main

import (
	"fmt"
	"sync/atomic"
)

type MySingleton struct {
	Value int
}

var instance *MySingleton

func GetSingleton() *MySingleton {
	if instance == nil {
		newInstance := &MySingleton{Value: 42}
		if atomic.CompareAndSwapPointer((*unsafe.Pointer)(unsafe.Pointer(&instance)), nil, unsafe.Pointer(newInstance)) {
			return newInstance
		}
	}
	return instance
}

func main() {
	singleton := GetSingleton()
	fmt.Println("Singleton Value:", singleton.Value)
}
```

在这个示例中，`GetSingleton` 函数首先检查 `instance` 是否为 `nil`，如果是，它创建一个新的实例，然后使用 `atomic.CompareAndSwapPointer` 来尝试原子地将新实例赋值给 `instance`。如果比较和交换成功，那么新的实例就会成为单例。这个过程是无锁的，因此不需要显式的锁来保护。

需要注意的是，`CompareAndSwap` 操作可以确保单例只会被创建一次，但如果的单例需要进行更复杂的初始化操作或需要确保初始化代码只运行一次，那么可能仍然需要使用锁来确保这些操作的线程安全性。在这种情况下，可以使用锁来包装初始化逻辑，而不是整个单例对象的创建。代码如下：

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

var (
    instance   *MySingleton
    initDone   int32
    initLock   sync.Mutex
)

type MySingleton struct {
    Value int
}

func InitSingleton() {
    if atomic.LoadInt32(&initDone) == 0 {
        initLock.Lock()
        defer initLock.Unlock()
        if initDone == 0 {
            instance = &MySingleton{Value: 42}
            atomic.StoreInt32(&initDone, 1)
        }
    }
}

func GetSingleton() *MySingleton {
    if atomic.LoadInt32(&initDone) == 0 {
        InitSingleton()
    }
    return instance
}

func main() {
    singleton := GetSingleton()
    fmt.Println("Singleton Value:", singleton.Value)
}
```

在这个示例中，使用原子操作来确保 `instance` 变量只会被初始化一次，实现了线程安全的单例模式。
# Go 并发相关

## Channel

### Channel 的分类

根据传输方向分类：

1. `chan T`: 可以接收和发送
2. `chan<- T`: 只可以发送
3. `<-chan T`: 只可以接收

根据通道容量分类：

```go
ch = make(chan int)     // 无缓冲通道
ch = make(chan int, 0)  // 无缓冲通道
ch = make(chan int, 3)  // 容量为 3 的缓冲通道
```

### ✨ 读写、关闭已关闭的 channel，会发生什么？

1. 读已关闭的 channel 时，如果 channel 内有元素，正常读；如果 channel 内没有元素，读到空值。
2. 读未关闭的空 channel 时，会报死锁的错误。
3. 写已关闭的 channel 时，会引发 panic。
4. 关闭已关闭的 channel 时，会引发 panic。

### Channel 是否并发安全？

Channel 是并发安全的，因为其在发送和接收时都加锁了。

- https://github.com/golang/go/blob/master/src/runtime/chan.go#L316
- https://github.com/golang/go/blob/master/src/runtime/chan.go#L648

## 锁

### Go 中提供了哪些锁？

- 互斥锁 (sync.Mutex)
  - `func (m *Mutex) Lock()`: 获取令牌（token，此过程也成为**上锁**）。
  - `func (m *Mutex) Unlock()`: 释放令牌。
- 读写互斥锁 (sync.RWMutex)
  - `func (rw *RWMutex) Lock()`: 写锁定
  - `func (rw *RWMutex) Unlock()`: 写解锁
  - `func (rw *RWMutex) RLock()`: 读锁定
  - `func (rw *RWMutex) RUnlock()`: 读解锁

### 互斥锁和读写互斥锁有哪些区别？

- 当一个 goroutine 获取读锁之后，其他的 goroutine 如果获取读锁会直接获得，如果获取写锁会等待。
- 当一个 goroutine 获取写锁之后，其他的 goroutine 无论获取读锁还是写锁都会等待。

### 互斥锁和读写互斥锁的使用场景是什么？

读写锁非常适合读多写少的场景，如果读和写的操作差别不大，读写锁的优势就发挥不出来。

## Go 中通道和锁的使用场景。

- 通道关注的是并发问题中的数据流动。适用于数据在多个协程中流动的场景。
- 锁关注的是数据不动，某段时间只给一个协程访问数据的权限。适用于数据位置固定的场景。

## Go 中 goroutine 如何退出

1. 通过向 goroutine 传入一个 channel 来通知 goroutine 退出。
2. 通过 context 上下文 (`context.Done()`) 通知 goroutine 退出。

## ✨ 进程、线程和协程的区别是什么？

进程 (Process) 是资源分配的最小单位，线程 (Thread) 是 CPU 调度的最小单位。协程 (Coroutine) 是用户态的轻量级线程。
一个进程可以包含多个线程，一个线程可以包含多个协程。

内核级线程的创建和销毁、调度需要陷入到内核态中，而协程可以认为完全在用户态创建、销毁和调度的。同时协程相比线程，占据的资源更加小。

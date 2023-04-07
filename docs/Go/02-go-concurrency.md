# Go 并发相关

## Channel

### Channel 的分类

1. `chan T`: 可以接收和发送
2. `chan<- T`: 只可以发送
3. `<-chan T`: 只可以接收

```go
ch = make(chan int)     // 无缓冲通道
ch = make(chan int, 0)  // 无缓冲通道
ch = make(chan int, 3)  // 容量为 3 的缓冲通道
```

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

### 互斥锁和读写互斥锁的区别

- 当一个 goroutine 获取读锁之后，其他的 goroutine 如果获取读锁会直接获得，如果获取写锁会等待。
- 当一个 goroutine 获取写锁之后，其他的 goroutine 无论获取读锁还是写锁都会等待。

#### 两者的使用场景

读写锁非常适合读多写少的场景，如果读和写的操作差别不大，读写锁的优势就发挥不出来。

## Go 中 goroutine 如何退出

1. 协程外 select 协程的 done channel 和 `time.After()`；协程内完成后通过 `select` 尝试发送 done channel。
2. 使用 context 通知协程结束。
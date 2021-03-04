### Context

```go
type Context interface {
  Deadline() (deadline time.Time, ok bool)
  Done() <-chan struct{}
  Value(key interface{}) interface{}
}
```

Context实际是为了解决各种调用之间的一致性问题，在请求到达时，生成一个Context对象，包含请求的一些基本信息，然后通过参数传递到后续的调用，包括生成的goroutine

```func Background() Context```用来创建跟节点，同时提供了4个函数用来创建子节点

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val interface{}) Context
```

上面4个函数，都会返回子Context和cancel函数，调用cancel()，会取消当前Context的所有子Context，这样用来控制Context的传递范围。取消的信号就是Context.Done()能读出东西。
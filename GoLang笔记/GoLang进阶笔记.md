[TOC]

## 一、defer使用注意事项

`defer`是Go语言提供的一种用于注册延迟调用的机制：让函数或语句可以在当前函数执行完毕后（包括通过return正常结束或者panic导致的异常结束）执行。 

在编程的时候，经常需要打开一些资源，比如数据库连接、文件、锁等，这些资源需要在用完之后释放掉，否则会造成内存泄漏。 

详细参考网络文章

[GoLang之轻松化解defer的温柔陷阱](https://www.cnblogs.com/qcrao-2018/p/10367346.html)

### 1、合理的使用defer

使用defer的时候需要对`err`和`对象`进行判断，以免造成隐形的panic

```go
f,err := os.Open(filename)
if err != nil {
    panic(err)
}

if f != nil {
    defer f.Close()
}
```


































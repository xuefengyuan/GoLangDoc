[TOC]

### 一、defer使用注意事项

`defer`是Go语言提供的一种用于注册延迟调用的机制：让函数或语句可以在当前函数执行完毕后（包括通过return正常结束或者panic导致的异常结束）执行。 

在编程的时候，经常需要打开一些资源，比如数据库连接、文件、锁等，这些资源需要在用完之后释放掉，否则会造成内存泄漏。 

详细参考网络文章

[GoLang之轻松化解defer的温柔陷阱](https://www.cnblogs.com/qcrao-2018/p/10367346.html)

#### 1、合理的使用defer

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

### 二、不要使用+和fmt.Sprinft操作字符串

#### 1、最有效的方式buffer

```go
strBuf := bytes.NewBufferString("")
for i := 0; i<1000; i++{
    strBuf.WriteString("test")
}
```

#### 2、使用strings.Join()

```go
a,b := "Hello","World"
for i:=0; i < 1000; i++{
    strings.Join([]string{a,b},"")
}
```

### 三、对于有固定字段的键值对，用临时Struct，不要用map[string]interface{}

> 用临时Struct在运行期间不需要动态分配内存，并且不需要像map那样去检查索引，所以速度快很多。



四、golang带证书访问接口

```go
// certf 证书路径
// key路径 
func HttpRequest(url, certf, keyf string) error {
	
	pool := x509.NewCertPool()

	caCrt, err := ioutil.ReadFile(certf)
	if err != nil {
		glog.Error("ReadFile err:", err)
		return err
	}
	pool.AppendCertsFromPEM(caCrt)

	cliCrt, err := tls.LoadX509KeyPair(certf, keyf)
	if err != nil {
		glog.Error("Loadx509keypair err:", err)
		return err
	}

	tr := &http.Transport{
		TLSClientConfig: &tls.Config{
			RootCAs:            pool,
			Certificates:       []tls.Certificate{cliCrt},
			InsecureSkipVerify: true,
		},
		DisableKeepAlives: true,
	}
	client := &http.Client{Transport: tr, Timeout: 60 * time.Second}

	req, err := http.NewRequest("GET", url, nil)
	resp, err := client.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	body, _ := ioutil.ReadAll(resp.Body)
	json.Unmarshal(body, rsp)
	return nil
}
```






























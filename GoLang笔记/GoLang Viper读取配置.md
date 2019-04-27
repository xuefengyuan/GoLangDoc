# GoLand Viper 读取配置

[TOC]



## 一、读取本地配置

```go

type Config struct {
    // 本地配置文件路径，不设置加载默认的
    ConfigFile string
    // 网络配置文件url，不设置加载默认的
    ConfigUrl string
}

// 加载本地配置文件
func (c *Config) loadFileConfig() error {

    if c.ConfigFile != "" {
        viper.SetConfigFile(c.ConfigFile) // 如果指定了配置文件，则解析指定的配置文件
        fmt.Println("config file path = ", c.ConfigFile)
    } else {

        // 如果没有指定配置文件，则解析默认的配置文件
        viper.AddConfigPath("../config/")
        viper.AddConfigPath("../../config/")
        viper.AddConfigPath("../../../config/")
        viper.AddConfigPath(".")
        // 配置文件名 对应的文件名为cofig.yaml
        viper.SetConfigName("config")
    }
    viper.SetConfigType("yaml") // 设置配置文件格式为YAML
    viper.AutomaticEnv()        // 读取匹配的环境变量
    appName := GetCurrentProcName()
    // 把小写字母转成大小
    appName = strings.ToUpper(appName)
    fmt.Println("app file name = ", appName)
    viper.SetEnvPrefix(appName) // 读取环境变量的前缀为appFileName 经过大写转换了
    replacer := strings.NewReplacer(".", "_")
    viper.SetEnvKeyReplacer(replacer)
    if err := viper.ReadInConfig(); err != nil { // viper解析配置文件
        return err
    }
	
    c.watchConfig()
    return nil
}

// 监控配置文件变化并热加载程序
func (c *Config) watchConfig() {
    viper.WatchConfig()
    viper.OnConfigChange(func(e fsnotify.Event) {
        log.Printf("Config file changed: %s", e.Name)
    })
}

// 获取当前执行的文件名
func GetCurrentProcName() string {
    return Last(os.Args[0], string(os.PathSeparator))
}

```





## 二、读取远程配置

```go
func loadRemoteConfig() error {
    cnf := api.DefaultConfig()
    // cof.Address 可以自定义，前提是consul支持远程访问
    cnf.Address = "127.0.0.1:8500"

    client, err := api.NewClient(cnf)
    if err != nil {
        return err
    }

    kv := client.KV()
    // 第二个参数为加载指定的key，如果为空，则加载目录下的全部key，
    keys, _, err := kv.Keys("/iocloud/conf/test", "testconfig1", nil)
    //keys, _, err := kv.Keys("/ioacloud/conf/default/global", "", nil)
    if err != nil {
        return fmt.Errorf("loadRemoteConfig failed,msg:%s.", err)
    } else {
        updateConfigByRemoteKeys(keys, kv)

    }

    return nil
}

func updateConfigByRemoteKeys(keys []string, kv *api.KV) {
    for _, key := range keys {
        pair, _, err := kv.Get(key, nil)
        if err != nil || pair == nil {
            continue
        }

        confitKey := Last(key, "/")

        if confitKey == "" {
            //fmt.Println("is directory ： ", key)
            continue
        }

        viper.SetConfigType("yaml")

        viper.ReadConfig(bytes.NewBuffer(pair.Value))
    }

}

func Last(str string, separator string) string {
    if str == "" {
        return ""
    }
    parts := strings.Split(str, separator)
    return parts[len(parts)-1]
}
```


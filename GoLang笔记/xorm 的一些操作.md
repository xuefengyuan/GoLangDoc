# Xorm 操作文档

[TOC]

## 一、Xorm安装

### 1、源码方式安装

#### 1.1、下载源码和第三方库

```shell
$ go get -u -v github.com/go-xorm/cmd/xorm

# 下载第三方依赖
$ go get -u -v github.com/go-xorm/xorm
# Mysql
$ go get -u -v github.com/go-sql-driver/mysql
# MyMysql
$ go get -u -v github.com/ziutek/mymysql/godrv
# Postgres
$ go get -u -v github.com/lib/pq
# SQLite
$ go get -u -v github.com/mattn/go-sqlite3
```

### 2、编译

>  进入到GOPATH\src\github.com\go-xorm\cmd\xorm 目录下编译

```shell
$ go build 
# windows下会生成xorm.exe，Linux下会生成xorm
# Windows不配置环境变量需要进入到编译目录执行，Linux不移动到bin目录下也是一样
$ xorm.exe help reverse
```

## 二、生成对应的go文件

### 1、mysql

```shell
$ xorm reverse mysql name:password@(ip:port)/xxx?charset=utf8 templates/goxorm
# 实际示例
$ xorm.exe reverse mysql root:hello1234@(127.0.0.1:3306)/alphago_runtime?charset=utf8 templates/goxorm
```

> 参数说明
>
> name:password@(ip:port)/xxx?charset=utf8
>
> name = 数据库用户名
>
> password = 数据库密码
>
> (ip:port) = 对应连接的数据库ip和端口
>
> xxx = 连接的数据库名称

### 2、PostgresSql

```shell
$ xorm.exe reverse  postgres "user=postgres password=ioa_cloud_xyz@168 host=10.123.24.5 port=25434 dbname=ioa_cloud sslmode=disable"  templates/goxorm
```

> 参数说明：
>
> user=postgres  
>
> password=ioa_cloud_xyz@168 
>
> host=10.123.24.5 
>
> port=25434 
>
> dbname=ioa_cloud s
>
> slmode=disable

## 三、GoLang连接数据库操作

### 1、连接MySql

```go
import (
	"httpsrv/config"
	_ "github.com/go-sql-driver/mysql"
	"github.com/go-xorm/core"
	"github.com/go-xorm/xorm"
	"github.com/golang/glog"
	_ "github.com/lib/pq"
)

const (
	maxIdleConns = 30
	maxOpenConns = 100
)

var MysqlDb *xorm.Engine

// InitMysql 初始化
func InitMysql() error {

	err := ConnectMysql(config.Conf.MysqlCfg)
	if err != nil {
		glog.Error("connect db failed, ", err)
	} else {
		glog.Info("connect db success.")
	}
	return err
}

//ConnectMysql 连接mysql
func ConnectMysql(mysqlconf config.MySQL) (err error) {
	if MysqlDb == nil {
		engine, err := xorm.NewEngine("mysql",
			fmt.Sprintf("%s:%s@tcp(%s:%s)/%s",
				mysqlconf.User,
				mysqlconf.Passwd,
				mysqlconf.Host,
				mysqlconf.Port,
				mysqlconf.DB))
		if err != nil {
			glog.Error("connect db failed, ", err)
			return err
		}
		engine.SetMaxIdleConns(maxIdleConns)
		engine.SetMaxOpenConns(maxOpenConns)
		engine.SetLogLevel(core.LOG_OFF)
		engine.ShowSQL(true)
		go keepDbAlived(engine)
		//return engine, err
		MysqlDb = engine
	}
	return nil
}
```

#### 1.1 、操作数据库

```go
// 查询,key为数据库对应的结构体，需要传入的是指针对象，Where里面的是对应的条件，可以使用?号占位符，可多个
MysqlDb.Table("table").Where("product=? and valid = 1", prod).Get(key)
// 只查询，不映射对象
MysqlDb.Table("table").Where("product = ? and valid = 1", prod).Exist()
// 查询指定字段，Cols里的表示需要查询的字段
MysqlDb.Table("table").Cols("name,pwd").Where("product=? and valid = 1", prod).Get(key)

// 更新
MysqlDb.Table("table").Update(key)
// 条件更新
MysqlDb.Table("table").Where("product = ? and valid = 1", prod).Update(key)

// 插入
MysqlDb.Table("table").Insert(key)

// 删除
MysqlDb.Table("table").Where("product = ? and valid = 1", prod).Delete(key)
```

### 2、连接PostgresSql

```go
import (
	. "common"	
	"github.com/spf13/viper"
	"public/base"
	"time"

	"github.com/go-redis/redis"
	"github.com/go-xorm/core"
	"github.com/go-xorm/xorm"
	"github.com/golang/glog"
	_ "github.com/lib/pq"
)
const (
	maxIdleConns = 30
	maxOpenConns = 100
)

var Db *xorm.Engine

// 连接Db
func InitDbConf() error {
	engine, err := ConnectDb(Conf.PgSQL)
	if err != nil {
		glog.Error("connect PostgresSql failed, ", err)
	} else {
		glog.Info("connect PostgresSql success.")
		Db = engine
	}

	return err
}

func ConnectDb(pgsqlconf PgSQL) (*xorm.Engine, error) {
	psqlInfo := fmt.Sprintf("host=%s port=%s user=%s password=%s dbname=%s sslmode=disable", pgsqlconf.Host, pgsqlconf.Port, pgsqlconf.User, pgsqlconf.Passwd, pgsqlconf.DB)
	
	engine, err := xorm.NewEngine("postgres", psqlInfo)
	if err != nil {
		glog.Error("connect db failed, ", err)
		return nil, err
	}
	if err != nil {
		glog.Error("connect db failed, ", err)
		return nil, err
	}
	engine.SetMaxIdleConns(maxIdleConns)
	engine.SetMaxOpenConns(maxOpenConns)
	engine.SetLogLevel(core.LOG_OFF)
	engine.ShowSQL(true)
	go keepDbAlived(engine)
	return engine, err
}
```

#### 2.1、操作数据库

操作数据库跟上面使用方式相同

```go
// 查询,key为数据库对应的结构体，需要传入的是指针对象，Where里面的是对应的条件，可以使用?号占位符，可多个
DB.Table("table").Where("product=? and valid = 1", prod).Get(key)
// 只查询，不映射对象
DB.Table("table").Where("product = ? and valid = 1", prod).Exist()
// 查询指定字段，Cols里的表示需要查询的字段
DB.Table("table").Cols("name,pwd").Where("product=? and valid = 1", prod).Get(key)

// 更新
DB.Table("table").Update(key)
// 条件更新
DB.Table("table").Where("product = ? and valid = 1", prod).Update(key)

// 插入
DB.Table("table").Insert(key)

// 删除
DB.Table("table").Where("product = ? and valid = 1", prod).Delete(key)
```

以上只是基础操作，后续有高级操作，就只一份示例了

## 四、Xorm的高级操作


























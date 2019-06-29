# 目录
# 1.1. 快速入门
  ## 1.1.1 安装
    go get github.com/suboat/sorm
  ## 1.1.2. 快速入门
```Golang
package main
import (
	"errors"
	"fmt"
	"github.com/suboat/sorm"
	_ "github.com/suboat/sorm/driver/mysql"
	"github.com/suboat/sorm/types"
)

// configDB 数据库连接参数
type configDB struct {
	DbName   string `json:"dbName"`   //
	User     string `json:"user"`     //
	Password string `json:"password"` //
	Host     string `json:"host"`     //
	Port     string `json:"port"`     //
}

// Student 学生表
type Student struct {
	StuID string `sorm:"size(36);primary" json:"stuId"`
	Name  string `sorm:"size(32);index" json:"name"`
	Age   int    `sorm:"size(23);index" json:"age"`
}

// 模型
var modelStu orm.Model

func main() {
	var (
		db     orm.Database
		dataDB = &configDB{
			DbName:   "mysql",
			User:     "business",
			Password: "business",
			Host:     "127.0.0.1",
			Port:     "33306",
		}
		err error
	)
	str := fmt.Sprintf(`{"user":"%s", "password": "%s", "host": "%s", "port": "%s", "dbname": "%s",
"sslmode": "disable","database":"business"}`, dataDB.User, dataDB.Password, dataDB.Host, dataDB.Port, dataDB.DbName)
	// 连接数据库
	if db, err = orm.New("mysql", str); err != nil {
		panic(err)
	}
	// 给模型赋值
	modelStu = db.Model("student2")

	// 创建表,初始化的时候会自动生成表，添加字段在表结构体添加后初始化会添加新的字段,为了安全起见,本驱动不提供删除字段方法
	if err = modelStu.Ensure(&Student{}); err != nil {
		panic(err)
	}

	// create
	var (
		s = new(Student)
	)
	s.Name = "Test"
	s.Age = 16
	s.StuID = types.NewUID() // new一个id

	// 创建数据
	if err = createStudent(s); err != nil {
		panic(err)
	}

	// 获取数据
	if _data, _err := getStudent(s.StuID); _err != nil {
		panic(_err)
	} else {
		fmt.Println(_data)
	}

	// 更新数据
	if _data, _err := updateStudent(s.StuID, nil, nil); _err != nil {
		panic(_err)
	} else {
		fmt.Println(_data)
	}

	// 删除数据
	if err = deleteStudent(s.StuID); err != nil {
		panic(err)
	}

}

// 创建
func createStudent(s *Student) (err error) {
	if s == nil {
		err = errors.New("student is nil")
		return
	}
	// 创建用户
	if err = modelStu.Objects().Create(s); err != nil {
		return
	}
	return
}

// 读取
func getStudent(id string) (ret *Student, err error) {
	ret = new(Student)
	// 获取用户
	if err = modelStu.Objects().Filter(orm.M{"stuID": id}).One(ret); err != nil {
		return
	}
	return
}

// 更新
func updateStudent(id string, age *int, name *string) (ret *Student, err error) {
	var (
		data  *Student
		_age  int
		_name string
	)
	if age != nil {
		_age = *age
	} else {
		_age = 0
	}
	if name != nil {
		_name = *name
	} else {
		_name = ""
	}
	// 更新数据
	if err = modelStu.Objects().Filter(orm.M{
		"stuID": id,
	}).UpdateOne(map[string]interface{}{
		"name": _name,
		"age":  _age,
	}); err != nil {
		return
	}

	// 获取最新数据返回
	if data, err = getStudent(id); err != nil {
		return
	}

	ret = data
	return
}

// 删除数据
func deleteStudent(id string) (err error) {
	if err = modelStu.Objects().Filter(orm.M{"stuID": id}).DeleteOne(); err != nil {
		return
	}
	return
}

```  
# 1.2. 连接数据库
## 1.2.1. 数据库连接  
MySQL:
``` Golang
package main

import (
	"github.com/suboat/sorm"
	_ "github.com/suboat/sorm/driver/mysql"
)

func main() {

	conn := `{"user":"business", "password": "business", "host": "127.0.0.1", "port": "33306", 
"database": "business", "sslmode": "disable"}`
	if db, err := orm.New(orm.DriverNameMysql, conn); err != nil {
		panic(err)
	} else {
		defer db.Close()
	}
}
```
PostgreSQL:
``` Golang
package main

import (
	_ "github.com/suboat/sorm/driver/pg"
	"github.com/suboat/sorm"
)
func main() {

	conn := `{"user":"business", "password": "business", "host": "127.0.0.1", "port": "65432", 
"database": "business","sslmode": "disable"}`
	if db, err := orm.New(orm.DriverNamePostgres, conn); err != nil {
		panic(err)
	} else {
		defer db.Close()
	}
}

```
Sqlite:
```Golang
package main

import (
	_"github.com/suboat/sorm/driver/sqlite"
	"github.com/suboat/sorm"
)
func main() {

	conn := `{"database":"data_sqlite/business.db"}`
	if db, err := orm.New(orm.DriverNameSQLite, conn); err != nil {
		panic(err)
	} else {
		defer db.Close()
	}
}
```
Mongo:
```Golang
package main

import (
	"github.com/suboat/sorm"
	_ "github.com/suboat/sorm/driver/mongo"
)

func main() {

	conn := `{"url":"mongodb://127.0.0.1:27017/", "db": "business"}`
	if db, err := orm.New(orm.DriverNameMongo, conn); err != nil {
		panic(err)
	} else {
		defer db.Close()
	}
}

```

## 1.2.2. 删除表
 ```Golang
 	db.Drop()
 ```
 ## 1.2.3. 创建表
 ```Golang
 	db.Ensure(&User{})
 ```
 # 1.3. 模型
 模型提供的方法
 ```Golang
// Model 模型定义
type Model interface {
	String() string                      // 表名
	Objects() Objects                    // 返回object对象
	ObjectsWith(opt *ArgObjects) Objects // 返回object对象

	// 通过tag来定义索引:
	// unique索引    Name string `sorm:"unique"`
	// index索引     Name string `sorm:"index"`
	// TODO: 全文索引 Name string `sorm:"text"`
	Ensure(st interface{}) error // 通过struct的tag来 添加字段,确认索引
	// 确认字段存在
	EnsureColumn(st interface{}) error
	// 确认索引存在
	EnsureIndex(index Index) error

	// 事务
	Begin() (Trans, error)                  // 事务开始
	BeginWith(opt *ArgTrans) (Trans, error) // 指定事务开始
	Commit(Trans) error                     // 阶段二提交
	Rollback(Trans) error                   // 回滚操作
	AutoTrans(Trans) error                  // 依据Trans内是否储存有错误来自动决定回滚或提交

	// 表的读写锁
	Lock()
	Unlock()
	RLock()
	RUnlock()

	// 直接执行数据库语句
	Exec(query string, args ...interface{}) (result Result, err error)
	// 删表
	Drop() error

	// other
	With(opt *ArgModel) Model // 返回一个新的model
}
 ```
 ## 1.3.1. 模型定义方法
 Key|Args|Example|Comment|
----|--------|--------|--------|
primary|-|sorm:"primary"|设置为主键|
serial|-|sorm:"serial"|自增数字主键|
uique|relative field(s)|sorm:"unique(username,email)"|(联合)唯一|
index|relative field(s)|sorm:"index(username,email)"|(联合)索引|
json|-|sorm:"json"|强制以数据库支持的json格式储存|
size|varchar size|sorm:"size(36)"|以36长度储存|
-|-|bson:",inline"|兼容mgo的结构体组合|  

修改字段的类型
```Golang
type User struct {
	Uid string `sorm:"size(36);primary" json:"uid"` // 主键
	CreateTime time.Time `sorm:"index" json:"createTime"` // 创建时间
	UpdateTime time.Time  `sorm:"index" json:"updateTime"` // 更新时间 
	Name  string `sorm:"size(32);index" json:"name"` // 长度为32
	Phone string   `sorm:"size(32);index" json:"phone"` // 唯一手机号码
	Username string `sorm:"size(16);unique"`   // 唯一用户名
	Age   int    `sorm:"size(23);index" json:"age"`
}
```
name 的长度改为64:
```Golang 
Name string   `sorm:"size(64);index"`
```
Sid和Name联合主键:
``` Golang 
Name string `sorm:"size(64),primary(sid)"` 
```  
## 1.3.2. 给模型赋值
```Golang
modelUser := db.Model("yourTableName")
```
## 1.3.3. 创建表
用user表为例,给modelUser赋值后,我们来创建数据表,
```Golang
modelUser.Ensure(&User)
```
如果需要添加字段,则在user结构体中添加后初始化即可  
`特别提示：` 本框架为避免程序员手残,特不提供删除字段方法,如需删除字段，请自己到数据库删除.  
# 1.4. 对象
Object: 返回Objects类型，可以执行创建、删除、更新、查询等操作，其提供的方法如下
```Golang
// Objects 数据对象操作
type Objects interface {
	// normal
	Filter(M) Objects                // 搜索结果
	Count() (int, error)             // 数目
	Limit(int) Objects               // 限制
	Skip(int) Objects                // 跳过
	Sort(...string) Objects          // 排序
	Meta() (*Meta, error)            // 摘要信息
	All(result interface{}) error    // 保存搜索结果至
	One(result interface{}) error    // 取一条记录
	Create(insert interface{}) error // 插入一条记录
	Update(record interface{}) error // 更改(输入struct或map) *** struct为覆盖更新，map为局部更新
	UpdateOne(obj interface{}) error // 只确保更改一条(输入struct或map)
	Delete() error                   // 删除
	DeleteOne() error                // 删除一条记录
	// 事务操作
	TLockUpdate(t Trans) error                 // 行锁
	TCount(t Trans) (int, error)               // 数目
	TAll(result interface{}, t Trans) error    // 保存搜索结果至
	TOne(result interface{}, t Trans) error    // 取一条记录
	TCreate(insert interface{}, t Trans) error // 插入一条记录
	TUpdate(obj interface{}, t Trans) error    // 更改
	TUpdateOne(obj interface{}, t Trans) error // 更改一条
	TDelete(t Trans) error                     // 删除
	TDeleteOne(t Trans) error                  // 删除一条记录
	// result
	GetResult() (Result, error) // 取返回结果
	// other
	With(opt *ArgObjects) Objects // 以新的日志级别运行
}
```
## 1.4.1. 创建记录
```Golang
	var user =new(User)
	uid:=types.NewUID()
	user.Uid=uid
	user.Age=18
	user.Name="大王叫我来巡山"
	user.Username="输了你赢了世界又如何"
	user.Phone="13733131131"
	// 创建 
	modelUser.Object().Create(data)
```
## 1.4.2 查找记录
### 1.4.2.1 查找一条记录
```Golang
	var data = new(User)
	modelUser.Object().Filter(orm.M{
	"uid":uid,
	}).One(data)
```
### 1.4.2.2 查找多条记录
```Golang
	var data =[]*User
	// 按最新创建时间排序，获取最多10条关于`"age"=18`的记录
	modelUser.Object().Sort("-createTime").Limit(10).Filter(orm.M{
	"age":18,
	}).All(&data)
```
`小提示:`  
如果不需要搜索特定字段的数据,请用for循环获取数据库全部数据；获取不是唯一字段的记录,请用All()方法获取.

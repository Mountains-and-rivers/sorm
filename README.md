# 目录
# 1.1. 快速入门
  ## 1.1. 安装
    go get github.com/suboat/sorm
  ## 1.2. 快速入门
```Goland
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

	// 创建表
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
  ## 1.2.2.  
# 

# 目录
# 1.1. 快速入门
  ## 1.1. 安装
    go get github.com/suboat/sorm
  ## 1.2. 快速入门
    package main

import (
	"fmt"
	"github.com/suboat/sorm"
	_ "github.com/suboat/sorm/driver/mysql"
)

type configDB struct {
	DbName   string `json:"dbName"`   //
	User     string `json:"user"`     //
	Password string `json:"password"` //
	Host     string `json:"host"`     //
	Port     string `json:"port"`     //
}
type Student struct {
	StuID string `sorm:"size(36);primary" json:"stuId"`
	Name  string `sorm:"size(32);index" json:"name"`
	Age   int    `sorm:"size(23);index" json:"age"`
}

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
	if db, err = orm.New("mysql", str); err != nil {
		panic(err)
	}
	modelStu = db.Model("student2")

	if err = modelStu.Ensure(&Student{}); err != nil {
		panic(err)
	}

	
}
# 1.2. 连接数据库
  ## 1.2.1. 数据库连接
  ## 1.2.2.  
# 

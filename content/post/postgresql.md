---
title: postgresql
#slug: chinese-test
date: 2019-09-05
categories:
- go
- postgresql
- beego
tags:
- beego
- postgresql
- go
thumbnailImagePosition: left
thumbnailImage: /img/postgresql.jpeg
---
beego 操作postgresql
<!--more-->

# beego 

beego 默认使用　"github.com/lib/pq"

自动建表,指定scheme

main.go

``` go
package main

import (
	"fmt"
	"github.com/astaxie/beego"
	"github.com/astaxie/beego/orm"
	_ "github.com/lib/pq"
	_ "monitor-go/routers"
	_ "monitor-go/models"
)

func init() {
	orm.RegisterDriver("postgres", orm.DRPostgres) // 注册驱动
	orm.RegisterDataBase("default", "postgres", beego.AppConfig.String("sqlconn"))

	o := orm.NewOrm()
  //设置scheme
	result, e := o.Raw("set search_path to monitor").Exec()
	if e != nil {
		panic(e)
	}

	fmt.Println(result.RowsAffected())
	orm.RunSyncdb("default", false, true)
}

func main() {
	if beego.BConfig.RunMode == "dev" {
		beego.BConfig.WebConfig.DirectoryIndex = true
		beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
	}

	orm.Debug = true
	o := orm.NewOrm()
	o.Using("default")

	beego.Run()
	//models.ExampleDB_Model()
}

```

rule.go

``` go
package models

import (
	"github.com/astaxie/beego/orm"
	"time"
)

type Rule struct {
	Id                 int
	rulesName          string
	CpuUsageUpperLimit float64
	CpuUsageLowerLimit float64
	MemUsageUpperLimit float64
	MemUsageLowerLimit float64
	PrometheusRuleName string `orm:"type(json)"`
	Action             int
	AssociatedService  string `orm:"type(json)"`
	Status             int
	CreateTime         time.Time `orm:"type(datetime)"`
	CreateUser         string
	UpdateTime         time.Time `orm:"type(datetime)"`
	UpdateUser         string
	ScaleMaxNum        int
	ScaleMinNum        int
	HpaName            string `orm:"type(json)"`
	//tableName          struct{} `sql:"monitor.rules_go"`
}

func (r *Rule) TableName() string {
	return "rules_go"
}

func init() {
	orm.RegisterModel(new(Rule))
}

```

# github.com/go-pg/pg

指定scheme

``` go
package models

import (
	"fmt"
	"github.com/go-pg/pg"
	"github.com/go-pg/pg/orm"
)

type User struct {
	Id        int64
	Name      string
	Emails    []string
  //指定scheme
	tableName struct{} `sql:"monitor.users"`
}

type Story struct {
	Id        int64
	Title     string
	AuthorId  int64
	Author    *User
	tableName struct{} `sql:"monitor.story"`
}

func (u User) String() string {
	return fmt.Sprintf("User<%d %s %v>", u.Id, u.Name, u.Emails)
}

func createSchema(db *pg.DB) error {
	for _, model := range []interface{}{&User{}, &Story{}} {
		err := db.CreateTable(model, &orm.CreateTableOptions{
			//Temp:        true,
			IfNotExists: true,
		})
		if err != nil {
			return err
		}
	}
	return nil
}

func ExampleDB_Model() {
	params := make(map[string]interface{})
	params["search_path"] = "monitor"
	//var ctx context.Context
	db := pg.Connect(&pg.Options{
		User:     "postgres",
		Password: "postgres",
		Database: "CloudPlatform",
		Addr:     "192.168.17.195:5432",
		OnConnect: func(conn *pg.Conn) error {
			_, err := conn.Exec("set search_path=?", params)
			if err != nil {
				panic(err)
			}
			return nil
		},
	})
	defer db.Close()

	err := createSchema(db)
	if err != nil {
		panic(err)
	}

	user1 := &User{
		Name:   "admin",
		Emails: []string{"admin1@admin", "admin2@admin"},
	}
	err = db.Insert(user1)
	if err != nil {
		panic(err)
	}

	err = db.Insert(&User{
		Name:   "root",
		Emails: []string{"root1@root", "root2@root"},
	})
	if err != nil {
		panic(err)
	}

	story1 := &Story{
		Title:    "Cool story",
		AuthorId: user1.Id,
	}
	err = db.Insert(story1)
	if err != nil {
		panic(err)
	}

	// Select user by primary key.
	user := &User{Id: user1.Id}
	err = db.Select(user)
	if err != nil {
		panic(err)
	}

	// Select all users.
	var users []User
	err = db.Model(&users).Select()
	if err != nil {
		panic(err)
	}

	// Select story and associated author in one query.
	story := new(Story)
	err = db.Model(story).
		Relation("Author").
		Where("story.id = ?", story1.Id).
		Select()
	if err != nil {
		panic(err)
	}

	fmt.Println(user)
	fmt.Println(users)
	fmt.Println(story)
	// Output: User<1 admin [admin1@admin admin2@admin]>
	// [User<1 admin [admin1@admin admin2@admin]> User<2 root [root1@root root2@root]>]
	// Story<1 Cool story User<1 admin [admin1@admin admin2@admin]>>
}

```

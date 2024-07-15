---
layout: post
title:  "Dabbling in postgres "
date:   2020-08-23 21:21:21 +0530
tags: [PostgreSQL,API]
---

Starting with a simple Ubuntu VM from vultr I am going to start to setup all of the packages needed to do stuff.


### Install pre reqs first

For this project you will need a few things like postgres and golang to be installed.

I am using ubuntu based systems for this walkthrough you can use the following commands:
```
sudo apt install  -y  postgresql golang
```

This should install everything you need to continue.

## Configuring the database
Make sure you have postgresql installed. You can test by using the ```psql``` command.

Generally you want to avoid using this with root. You want to use the postgres user to login to the database.

```
su - postgres
```

you can use the following above to switch to the postgres user as root.

from here use the ```psql``` command to start using the postresql interface.


```
postgres@vultr:~$ psql
psql (16.3 (Ubuntu 16.3-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# 

```

Once your here now you can start configuring the database.

Next lets create the database:
```
CREATE DATABASE todoapp;
```

Now we will come back to this open a new terminal connected to your host.

## Go project structure
your project structure should look something like this:
```
todo_api/
├── main.go
├── handlers.go
├── models.go
├── database.go
├── go.mod
└── go.sum

```

After this we are going to make the file needed to make our project. You can touch all of these files and we will add the code after. 


Let's init the go modules:
```
go mod init todo_api
```

Now let's start editing our files let's start with

go.mod:
```
module todo_api

go 1.16

require(
	github.com/gin-gonic/gin v1.8.1
	github.com/jinzhu/gorm v1. 9.16
	github.com/jinzhu/gorm/dialects/postgres v1.9.16
)
```

Next files is

database.go:
```
package main

import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/postgres"
    "log"
)

var db *gorm.DB

func initDatabase() {
    var err error
    db, err = gorm.Open("postgres", "host=localhost user=postgres dbname=todoapp sslmode=disable password=yourpassword")
    if err != nil {
        log.Fatal(err)
    }

    db.AutoMigrate(&ToDoItem{})
}
```

models.go:
```
package main

import "github.com/jinzhu/gorm"

type ToDoItem struct {
    gorm.Model
    Title       string `json:"title"`
    Description string `json:"description"`
    Done        bool   `json:"done"`
}
```

handlers.go
```
package main

import (
    "github.com/gin-gonic/gin"
    "net/http"
)

func getToDos(c *gin.Context) {
    var todos []ToDoItem
    db.Find(&todos)
    c.JSON(http.StatusOK, todos)
}

func createToDo(c *gin.Context) {
    var todo ToDoItem
    if err := c.ShouldBindJSON(&todo); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    db.Create(&todo)
    c.JSON(http.StatusCreated, todo)
}

func updateToDo(c *gin.Context) {
    var todo ToDoItem
    id := c.Param("id")

    if err := db.Where("id = ?", id).First(&todo).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "ToDo not found"})
        return
    }

    if err := c.ShouldBindJSON(&todo); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    db.Save(&todo)
    c.JSON(http.StatusOK, todo)
}

func deleteToDo(c *gin.Context) {
    var todo ToDoItem
    id := c.Param("id")

    if err := db.Where("id = ?", id).First(&todo).Error; err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": "ToDo not found"})
        return
    }

    db.Delete(&todo)
    c.Status(http.StatusNoContent)
}
```

main.go
```
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    initDatabase()
    defer db.Close()

    router := gin.Default()

    router.GET("/todos", getToDos)
    router.POST("/todos", createToDo)
    router.PUT("/todos/:id", updateToDo)
    router.DELETE("/todos/:id", deleteToDo)

    router.Run(":8080")
}
```


Now that you have everything  you can go ahead and run/build your application with the following:

``go run main.go database.go models.go handlers.go``

and you should get something like this as a repsonse:

`[GIN-debug] Listening and serving HTTP on :8080

If you get this then you are ready to start using your todo app.

You can create a new entry using:
`curl -X POST -H "Content-Type: application/json" -d '{"title":"Test"}' http://localhost:8080/todos
`
You can also Get all entries using:
`curl http://localhost:8080/todos
`
You can also update a entry with:
``curl -X PUT -H "Content-Type: application/json" -d '{"title":"Updated ToDo","done":true}' http://localhost:8080/todos/[ID]`

just make sure to replace the ID with the proper ID.

and finally to delete a entry use:
`curl -X DELETE http://localhost:8080/todos/[ID]

remember you change the ID.

This concludes the Project you can add many features to this and make a few things for this.


`

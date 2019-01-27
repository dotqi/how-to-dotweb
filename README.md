## DOTWEB  使用心得  
项目地址：https://github.com/devfeel/dotweb  
获取方式：go get -u github.com/devfeel/dotweb  

1、简单部署  
``` go
package main
import (
	"fmt"
	"github.com/devfeel/dotweb"
)
func main() {
	app := dotweb.New()
	// 设置路由 输出字符串 Hello Dotweb
	app.HttpServer.GET("/", func(ctx dotweb.Context) error {
		return ctx.WriteString("Hello Dotweb")
	})
	fmt.Println("dotweb.StartServer => 8080")
	//开启服务 端口号
	err := app.StartServer(8080)
	fmt.Println("dotweb.StartServer error => ", err)
}
```
2、启用TLS  

3、文件服务  
``` go
package main
import (
	"fmt"
	"github.com/devfeel/dotweb"
)
func main() {
	app := dotweb.New()
	// 开启显示文件列表功能 默认关闭
	app.HttpServer.SetEnabledListDir(true)
	// 设置文件服务支持中间件 默认关闭
	// 开启后，可同中间件配合，让文件服务支持跨域，或其他需求比如权限控制
	app.HttpServer.SetEnabledStaticFileMiddleware(true)
	// 默认文件服务器注册方式只处理GET请求
	// 访问地址为 http://127.0.0.1:8080/static
	app.HttpServer.ServerFile("/static/*", "D:/GoWork/GoTest/view")
	// 注册中间件
	//app.HttpServer.ServerFile("/view/*", "D:/GoWork/GoTest/view").Use(NewSessionAuth())
	// 注册不同请求方式
	app.HttpServer.RegisterServerFile(dotweb.RouteMethod_GET, "/get/*filepath", "D:/GoWork/GoTest/view")
	app.HttpServer.RegisterServerFile(dotweb.RouteMethod_POST, "/post/*filepath", "D:/GoWork/GoTest/view")
	//开启服务 端口号
	fmt.Println("dotweb.StartServer => 8080")
	err := app.StartServer(8080)
	fmt.Println("dotweb.StartServer error => ", err)
}
```

4、支持跨域
    请求跨域时，需要中单件支持。如果自行设置中单件，可参考cors代码。  
    获取地址：go get -u github.com/devfeel/middleware  
    引用：import "github.com/devfeel/middleware/cors"  

``` go
package main
import (
    "fmt"
    "github.com/devfeel/dotweb"
    "github.com/devfeel/middleware/cors"
)
func main() {
    app := dotweb.New()
    // app注册中间件，设置cors选项
    option := cors.NewConfig().UseDefault()
    app.Use(cors.Middleware(option))

    // 开启所有路由的OPTIONS请求支持 默认不开启
    // AJAX跨域请求会发送OPTIONS请求。
    // 也可自行设置OPTIONS请求路由
    app.HttpServer.SetEnabledAutoOPTIONS(true)

    // 设置路由 输出字符串 Hello Dotweb
    app.HttpServer.GET("/", func(ctx dotweb.Context) error {
        method := ctx.Request().Method
        return ctx.WriteString("Hello Dotweb\n" + "Method:" + method)
    })
    // 设置路由 输出字符串 Hello Dotweb
    app.HttpServer.POST("/", func(ctx dotweb.Context) error {
        method := ctx.Request().Method
        return ctx.WriteString("Hello Dotweb\n" + "Method:" + method)
    })

    //开启服务 端口号
    fmt.Println("dotweb.StartServer => 8080")
    err := app.StartServer(8080)
    fmt.Println("dotweb.StartServer error => ", err)
}
```

5、Session  
	// 开启SESSION 默认关闭  
	app.HttpServer.SetEnabledSession(true)  
	ctx.Session().Set(Key, Value)  
	ctx.Session().Get(Key)  
	ctx.Session().Remove(Key)  

6、Cookie  
7、分组路由  

8、中间件Middleware   
	中间件注册方法：Use()  
	支持中间件注册点：App、Group、RouterNode
	ServerFile也支持注册中间件，方法同2。
	中间件：输出SessionID
``` go
package main
import (
	"fmt"
	"github.com/devfeel/dotweb"
)

func main() {
	app := dotweb.New()
	// App注册中间件
	app.Use(NewSessionAuth())
	// 开启SESSION
	app.HttpServer.SetEnabledSession(true)
	// 设置路由 输出字符串 Hello Dotweb
	app.HttpServer.GET("/", func(ctx dotweb.Context) error {
		method := ctx.Request().Method
		return ctx.WriteString("Hello Dotweb\n" + "Method:" + method)
	})

	//开启服务 端口号
	fmt.Println("dotweb.StartServer => 8080")
	err := app.StartServer(8080)
	fmt.Println("dotweb.StartServer error => ", err)
}

// SessionAuth 结构体
type SessionAuth struct {
	dotweb.BaseMiddlware
}

// Handle 处理程序
func (m *SessionAuth) Handle(ctx dotweb.Context) error {
	fmt.Println("SessionID = ", ctx.SessionID(), " RequestURI = ", ctx.Request().RequestURI)
	return m.Next(ctx)
}

// NewSessionAuth New
func NewSessionAuth() *SessionAuth {
	sAuth := new(SessionAuth)
	return sAuth
}
```

# xiaoxue-backend 晓学后端  
1、采用go语言提供restful格式的api接口。    
2、在xiaoxue-doc 工程下，我们已经通过powerdeginer制作了项目的概念模型，并通过powerdeginer生成了mysql脚本文件。    
3、根据myslq的脚本文件，为了以后应对高并发、大数据量，我做了数据库分库处理，分别建了以用户为维度的用户表、用户收藏表；以文章为维度的文章表，文章分类表，文章表，文章评论表，文章喜欢表，文章阅读表，文章标签表，标签表；以消息为维度的消息表；以私信为唯独的私信表。   
4、确保你的环境安装了go语言环境并安装了beego模块。   
5、在命令行工具中输入以下命令完成api的创建：bee api xiaoxue_article-api -conn="*:123456@tcp(115.29.*.*:3306)/xiaoxue_article"   
6、beego介绍
beego是一个Golang实现的开源Go应用开发框架，他可以用来快速开发 API、Web 及后端服务等各种应用，是一个 RESTful的框架，主要设计灵感来源于tornado、sinatra和flask这三个框架，但是结合了Go本身的一些特性（interface、struct 嵌入等）而设计的一个框架。

Beego Framework:
一个使用 Go 的思维来帮助您构建并开发 Go 应用程序的开源框架
beego简介

beego安装， bee命令
安装beego

#go get github.com/astaxie/beego
安装bee工具，bee工具是一个为了协助快速开发beego项目而创建的项目，可以通过bee快速创建项目、实现热编译、开发测试以及开发完之后打包发布的一整套从创建、开发到部署的方案。

#go get github.com/beego/bee
bee命令默认安装在$GOPATH/bin下，把这个路径添加到PATH中。

实践中主要用到bee的三个子命令：api，run，pack。

api命令用来创建api应用，生成默认beego api应用框架。

# bee api snmpcheck
# tree snmpcheck/
snmpcheck/
├── conf
│   └── app.conf
├── controllers
│   ├── object.go
│   └── user.go
├── docs
│   └── doc.go
├── main.go
├── models
│   ├── object.go
│   └── user.go
├── routers
│   └── router.go
└── tests
└── default_test.go
其中，routers/router.go是路由的相关配置，controllers目录下存放各个api路由下相关的控制器。

run命令用来编译运行beego工程，并通过fsnotify监控源码的改动，实现热编译，开发过程中就可以实时的看到项目修改之后的效果。

# bee run
bee   :1.4.1
beego :1.6.1
Go    :go version go1.5.1 linux/amd64

[INFO] Uses 'snmpcheck' as 'appname'
[INFO] Initializing watcher...
[TRAC] Directory(/home/lab/src/snmpcheck/controllers)
[TRAC] Directory(/home/lab/src/snmpcheck)
[TRAC] Directory(/home/lab/src/snmpcheck/models)
[TRAC] Directory(/home/lab/src/snmpcheck/routers)
[TRAC] Directory(/home/lab/src/snmpcheck/tests)
[INFO] Start building...
[SUCC] Build was successful
[INFO] Restarting snmpcheck ...
[INFO] ./snmpcheck is running...
[parser.go:61][I] /home/lab/src/snmpcheck/controllers no changed 
[parser.go:61][I] /home/lab/src/snmpcheck/controllers no changed 
[asm_amd64.s:1696][I] http server Running on :7070
pack命令用来发布应用的时候打包。

# bee pack
app path: /home/lab/src/snmpcheck
build snmpcheck
GOOS linux GOARCH amd64
build success
exclude relpath prefix: .
exclude relpath suffix: .go:.DS_Store:.tmp
file write to `/home/lab/src/snmpcheck/snmpcheck.tar.gz`
打包完的tar包中有应用的可执行文件和配置文件，部署时直接上传这个tar包即可。

# tar -tf snmpcheck.tar.gz 
snmpcheck
conf/app.conf
restful路由
beego的路由设置比较灵活，包括固定路由，正则匹配路由，以及通过go反射机制实现的自动路由（可能会导致性能损耗，不推荐使用这种路由设置方式）和注解路由。

实践中使用比较方便的注解路由方式。注解路由的使用：

首先在router中用namespace方式注册控制器。这里在/v1/user下，导入UserController控制器。

ns := beego.NewNamespace("/v1",
    ...
    beego.NSNamespace("/user",
        beego.NSInclude(
            &controllers.UserController{},
        ),
    ),
    ...
)
beego.AddNamespace(ns)
在控制器中对应方法上用注解方式注册路由。

// @Title logout
// @Description Logs out current logged in user session
// @Success 200 {string} logout success
// @router /logout [get]
func (u *UserController) Logout() {
    u.Data["json"] = "logout success"
    u.ServeJSON()
}
注解路由使用关键字***@router***。
这里"@router /logout [get]"注册了"Get /v1/user/logout -> UserController.Logout()"这样的路由。
如果beego运行在dev模式（可以在conf中配置），routers目录下会生成路由经过解析后的结果commentsRouter.go，调试时可以作为参考。

进程内监控
beego提供了应用信息的监控和展示，可以查看实时信息比如qps，健康状况，程序性能相关（goroutine，gc，cpu等），可以查看静态信息比如路由配置，conf配置信息，过滤器信息，还可以查看和触发任务。
进程监控默认是关闭的，可以通过简单的配置中打开，很方便：

EnableAdmin = true
AdminHttpAddr = 0.0.0.0 #默认监听地址是localhost
AdminHttpPort = 8088
这样，应用启动后，会在8088端口提供监控展示服务。


自动化文档
beego通过swagger和内部的注释解析能够实现自动文档的效果，使用方法：

routers/router.go中路由只能使用namespace+Include的写法，而且只支持二级解析，一级版本号，二级分别表示应用模块。

routers/router.go文件中设置全局的应用信息。注意，必须写在文件顶部。

注释的格式（每个字段的含义可以参照Auto Docs）：

// @Title login
// @Description Logs user into the system
// @Param   username        query   string  true        "The username for login"
// @Param   password        query   string  true        "The password for login"
// @Success 200 {string} login success
// @Failure 403 user not exist
// @router /login [get]
func (u *UserController) Login() {
    username := u.GetString("username")
    password := u.GetString("password")
    if models.Login(username, password) {
        u.Data["json"] = "login success"
    } else {
        u.Data["json"] = "user not exist"
    }
    u.ServeJSON()
}
在配置文件中打开自动文档配置：

EnableDocs = true
启动时添加自动文档参数：

bee run -gendoc=true
满足以上配置，beego会自动解析控制器中的注释，启动swagger服务，并在/docs接口上提供已生成好的json字串。
访问swagger服务并在swagger中访问/docs接口，即可看到接口的文档，同时也可以对接口进行测试。



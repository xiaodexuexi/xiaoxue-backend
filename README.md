# xiaoxue-backend 晓学后端  
1、采用go语言提供restful格式的api接口。    
2、在xiaoxue-doc 工程下，我们已经通过powerdeginer制作了项目的概念模型，并通过powerdeginer生成了mysql脚本文件。    
3、根据myslq的脚本文件，为了以后应对高并发、大数据量，我做了数据库分库处理，分别建了以用户为维度的用户表、用户收藏表；以文章为维度的文章表，文章分类表，文章表，文章评论表，文章喜欢表，文章阅读表，文章标签表，标签表；以消息为维度的消息表；以私信为唯独的私信表。   
4、确保你的环境安装了go语言环境并安装了beego模块。   
5、在命令行工具中输入以下命令完成api的创建：bee api xiaoxue_article-api -conn="*:123456@tcp(115.29.*.*:3306)/xiaoxue_article"           
6、通过bee run启动，如果想同时生成文档，则需要在beego的配置文件中加入：EnableDocs = true，启动命令为：bee run -gendoc=true。   
7、我们通过swagger来展示我们的文档，可以去官方地址https://github.com/swagger-api/swagger-ui下载Swagger-UI, 把该项目dist目录下的内容拷贝到项目的swagger目录下，修改index.html中的默认的url为(http://127.0.0.1:8080/swagger.json)    
8、通过机上步骤，我们开发好了自己的接口和部署好了swagger接口测试环境，下面就是具体的业务逻辑了。

# go-swagger

## 1. Swagger 规范

swagger的官方网址为 [swagger.io](http://swagger.io/),

官方介绍 
> 
> THE WORLD'S MOST POPULAR API FRAMEWORK
> 
> Swagger is a powerful open source framework backed by a large ecosystem of tools that helps you design, build, document, and consume your RESTful APIs.” 
> 

Swagger规范和框架是Wordnik公司(世界上最大的在线英文辞典,后更名为Reverb)驱动作为内部使用工具进行开发，项目开始于2010年。在2015年SmartBear公司从Reverb科技公司后去了开源Swagger API规范。

2015年11月，负责维护Swagger规范的SmartBear公司，宣称在Linux Foundation组织的资助下成了一个新的组织- Open API Initiative, 成员包括Google/IBM/Microsoft等。同时，SmartBear也将Swagger的规范捐赠给了该组织。[1]

Swagger是一套完整的规范，并提供了完善的工具支撑，方便生成接口文档，可以用来非常快捷和方便实现Restful API。框架为创建 JSON 或 YAML（JSON 的一个人性化的超集）格式的 RESTful API 文档提供了Swagger规范(后称作Open API)。

2010-2014年间分别发布过1.0、1.1与 1.2，每个版本都是较前版本小幅度的改进。在Swagger的开放工作小组（自2014年5月成立）的不懈努力下，Swagger 2.0终于在2014年9月正式发布，2.0版本只是对于规范进行了一次较大的改动。[2]

目前主流的语言都支持了对于swagger的支持，语言支持列表参见：[Tools and Integrations](http://swagger.io/open-source-integrations/)

> 一些 Swagger 编辑工具可帮助您轻松地创建 API 文档，确保它们遵守 OpenAPI 规范。举例而言，通过使用 [Swagger Editor](http://editor.swagger.io/#/)，您可以创建或导入 API 文档，并在一个交互式环境中浏览它。右侧的显示窗格显示了格式化的文档，反映了您在左侧窗格中的代码编辑器中执行的更改。代码编辑器指出了所有格式错误。您可以展开和折叠每个窗格。
以下是您导入 leads.yaml 定义后的 Swagger Editor UI 外观：[3]

![png](http://www.ibm.com/developerworks/cn/web/wa-use-swagger-to-document-and-define-restful-apis/image001.png)

除了[Swagger Editor](http://editor.swagger.io/#/)外，[swagger-ui](https://github.com/swagger-api/swagger-ui)也是比较出名的提供在线文档接口的优秀工具。


## 2. go-swagger

Github地址：[go-swagger/go-swagger](https://github.com/go-swagger/go-swagger)，项目由VMware资助， 官方文档 [goswagger.io](https://goswagger.io/)


### 2.1 安装

由于0.5.0和0.6.0版本出现了一些pkg兼容性的问题，目前最稳定版本为0.8.0， **brew install** 安装的版本为0.5.0，因此推荐安装采用Static binary的方式进行安装。

	$ latestv=$(curl -s https://api.github.com/repos/go-swagger/go-swagger/releases/latest | jq -r .tag_name)
	$ curl -o /usr/local/bin/swagger -L'#' https://github.com/go-swagger/go-swagger/releases/download/$latestv/swagger_$(echo `uname`|tr '[:upper:]' '[:lower:]')_amd64
	$ chmod +x /usr/local/bin/swagger
	
然后安装go-swagger的源码：
	
	$ go get -u github.com/go-swagger/go-swagger

快速入门的教程参见：[Tutorials](https://goswagger.io/tutorial/todo-list.html)，完整的to-do-list server代码参见源码 [github.com/go-swagger/go-swagger/examples/tutorials/todo-list/server-complete/cmd/todo-list-server](https://github.com/go-swagger/go-swagger/tree/master/examples/tutorials/todo-list/server-complete)，然后就可以生成想代码进行测试。

 	$ cd tutorials/todo-list/server-complete/cmd
	$ go build && ./main    # 备注 go build过程中会出现提示安装更多的开源库，使用go get安装即可
	
如果生成的访问地址为： http://127.0.0.1:9090/，则可以使用 http://127.0.0.1:9090/docs 进行API文档测试，默认文档样式为 Redoc 格式，如果使用 Swagger 格式则配合[swagger-ui](https://github.com/swagger-api/swagger-ui)则方便进行 UI 页面测试。

从官方下载 swagger-ui， 直接运行 swagger-ui/dist/index.html，然后输入 http://127.0.0.1:9090/swagger.json 即可访问。

由于存在[跨域问题](https://github.com/go-swagger/go-swagger/issues/481), 因此需要修改server端 restapi/configure_todo_list.go 代码：

	import "github.com/rs/cors"
	
	func setupGlobalMiddleware(handler http.Handler) http.Handler {
		handleCORS := cors.Default().Handler    // 修改成cores的默认放通
		return handleCORS(handler)
	}

然后重新在 swagger-ui 的界面中输入即可。

![样例图](http://www.do1618.com/wp-content/uploads/2016/12/to_do_list_api.png)

## 参考

1. [OpenAPI Specification](https://en.wikipedia.org/wiki/OpenAPI_Specification)
2. [通过Swagger进行API设计，与Tony Tam的一次对话](http://www.infoq.com/cn/articles/swagger-interview-tony-tam)
3. [使用 Swagger 文档化和定义 RESTful API](http://www.ibm.com/developerworks/cn/web/wa-use-swagger-to-document-and-define-restful-apis/index.html)
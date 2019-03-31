title: 项目结构介绍
---
## 目录结构
![enter image description here](https://thumbnail10.baidupcs.com/thumbnail/0c848b1cb6c406af8c1d1eeede5c067c?fid=1117135785-250528-555699137109538&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-dFl8JnDr74mf8F9fBnhMlrnTVIA%3d&expires=8h&chkbd=0&chkv=0&dp-logid=1964447948469035519&dp-callid=0&time=1553587200&size=c1920_u1080&quality=90&vuk=1117135785&ft=image&autopolicy=1)

接下来，我将一一介绍每个文件以及文件夹的定义
### components 

公共视图文件以及公共模块脚本的存放。

受项目业务场景限定，基本上一级界面的详情模块，在很多地方都需要共用。按照以往的模式开发，一个界面资源在渲染时就加载，那么，如果一个模块很大程度上复用了其它模块的视图文件时，这个模块就会变得极其庞大和难以维护，同时很大程度上浪费请求资源以及模块的耦合性会变得很难维护。

以往住建部的项目，没有前后端分离之前，采用的是iframe 层层嵌套，这种方式一定程度上解决了模块共用问题，但也很大程度上照成了资源请求浪费与界面加载的性能问题，交互上增加了不少的复杂性也很难让前端正规军接受。所以，我们将每个模块有可能公用到的地方，都提到与视图文件views同级的components文件夹底下。

### controller
这个文件原本是layuiadmin里面的模块代码存放位置，本项目采用的是sea.js管理模块依赖与编写，所以这个项目目前暂时没有其它用途，如果有些layui模块脚本，放置此处。
### font
项目的字体图标文件，主要以iconfont图标为主
### img 
项目依赖的图片文件
### js
存放全局配置文件以及公用函数的文件夹。
其中包含：
- app.js 这是sea.js的启动程序。配置依赖加载与统一公用模块的执行。
- config.js layadmin的全局配置
- common、plugins文件夹，存放公用资源模块与插件模块

### json
存放静态测试数据
### style
存放样式文件。scss文件统一编译app.scss 其它模块则在app.scss中加载即可、
### views
视图模板文件的存放目录，依据项目的菜单建目录，具体里面的写法可以查阅[模块快速搭建]()
### index.html
单页项目启动页，包含样式加载与sea.js初始化配置
### index.js
layuiadmin的单页启动程序主入口。用于加载依赖插件、默认启动页、路由拦截、监听hash等操作。
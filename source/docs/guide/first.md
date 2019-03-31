title: 第一个模块开发
---
## 前言
在开始开发模块之前，确保你已经熟读了[html规范-代码规范](/public/docs/html/code.html)、[html规范-文件模板规范](/public/docs/html/note.html)
### 开始你的模块开发
在[项目结构介绍](/public/docs/common.html)中，已知我们需要创建一级界面，需要再views目录底下创建文件。之前尝试过很多种方式去做模块的js管理，有统一放置controller底下进行合并压缩，按视图文件的目录结构在controller底下建立一样的结构做映射。 也尝试过将js放置模板文件本身处理当前模板需要的交互。 前者缺点比较明显，当业务系统足够复杂，开发过程中，文件结构过多，维护起来就比较费力了。后者虽然模板片段与js都在同一个文件做维护，但耦合性太高，资源重复加载，函数重复的销毁去注册，在复杂系统中，无形给浏览器增加没必要的负载。

所以，在模块开发过程中，我们借鉴vue的模板写法，将视图文件与js放置同一路由路径底下。通过sea.js的模块缓存机制，既解决开发与维护问题，同时解决资源没必要的重复加载。

好了，废话不多说，让我们尝试开始第一个模块的开发

### 创建视图目录
以企业端-业务办理菜单为例：
在目录views下创建businessDealt目录为业务办理视图文件目录
```
views ==> businessDealt
```
其中业务办理菜单底下包含：资质申请、安全生产许可证申请、证书变更申请、历史办件库、注册人员锁定等二级菜单 
![](https://thumbnail10.baidupcs.com/thumbnail/83d325ed46062a2263e863ca67576f7b?fid=1117135785-250528-285817741304576&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-nyh52UZOAOXaMWriYdELIfi7%2ff4%3d&expires=8h&chkbd=0&chkv=0&dp-logid=2010931734961767558&dp-callid=0&time=1553760000&size=c1920_u1080&quality=90&vuk=1117135785&ft=image&autopolicy=1)
在businessDealt目录底下分别创建对应的模板目录
```
 businessDealt
    - qualificatApply
    - safeLicenseApply
    - changeRequest
    - registryLock
    - archive
```
### 创建对应视图文件
在创建好对应的目录结构之后，我们将对应模块的视图目录底下分别建index.html作为当前路由视图的模板文件，index.js作为当前路由视图的js执行脚本 
以资质申请为例：
> qualificatApply ==> index.html

```html
<title>资质申请</title>
<span class="layui-breadcrumb" lay-separator="&gt;">
    <a href="#/">首页</a>
    <a>业务办理</a>
    <a>资质申请</a>
</span>
<div class="layui-row layui-col-space15 qualification">
  <div class="layui-col-md12">
      <!-- 填充内容 -->
      <table class="layui-hide" lay-filter="qualificatApply" id="qualificatApply"></table>
  </div>
</div>
<script>
    layui.use(['admin','element', 'form', 'table', 'laydate', 'upload','tree'], function () {
    var $ = layui.$,
      element = layui.element,
      form = layui.form,
      table = layui.table,
      laydate = layui.laydate,
      view = layui.view,
      tree = layui.tree,
      router = layui.router();

    seajs.use('./views/businessDealt/qualificatApply/index', function (e) {
      new e()
    })
  })
</script>
```

其中title作为当前路由页的title, layui-breadcrumb部分作为面包屑渲染的结构（<font color=red>此处为一级模板界面必须添加的组成部分</font>）

layui-row 为视图主体部分，编写主体内容代码

script标签里的js代码，特别<font color=red>需要注意</font>的是，当前视图文件所需的layui组件都必须在当前模板中引入（<font color=green>防止当前路由页面刷新，造成layui组件引用失效</font>）

当前模块的js代码是如何加载执行的呢？ 看script标签里面的seajs加载
```js
    seajs.use('./views/businessDealt/qualificatApply/index', function (e) {
      new e()
    })
```
由于受限seajs的baseUrl配置的限制，我们将baseUrl指向的是当前项目的根目录（不建议在其它js模块中做更改，否则会路由切换会造成资源路径混乱，模块中的依赖关系难以维护），暂时只能通过从跟目录逐级往下找的方式去加载当前路由视图的js模块(<font color=blue>待解决...</font>)

那么，模块中的代码index.js该如何编写呢？看[路由视图的js编写](/public/docs/guide/first_js.html)
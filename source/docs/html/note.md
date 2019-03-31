title: 文件模板规范
---
## 命名遵循标准
模板的命名使用与业务相关联的名称作为文件名 例如详情页使用`info.html` 详情页如果还有涉及到模板，则命名使用统一为`info_{{业务名称}}`，例如详情有个tab栏，其中有一块视图名为基本信息，则模板名称为`info_base.html`。命名是请用<font color=red>驼峰式命名</font>

### 项目中模板地址存放
针对于本项目特殊的业务场景，在不用栏目中，有可能需要调用其它栏目的详情页，这时，相比以往项目经验，类似详情页的模板文件以及执行的模块代码就必须抽离到公共的资源文件夹底下。同时保证视图文件以及模块的js文件能按需加载并执行。

针对于这样的业务场景，我们将公共的资源文件 (如二级界面) 提取到与视图文件views同级的components目录底下。同时以具体的模块一级页面的名称作为当前模块的共用资源文件夹。存放html模板与模板对应的js模块。例如：
![enter image description here](https://thumbnail10.baidupcs.com/thumbnail/6b4a582f9da8bb260f522e59ea6a923f?fid=1117135785-250528-719710553653738&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-ToSuoNWgjfhpNfa7kSZi9SVdhMs%3d&expires=8h&chkbd=0&chkv=0&dp-logid=1959722739227705513&dp-callid=0&time=1553569200&size=c1920_u1080&quality=90&vuk=1117135785&ft=image&autopolicy=1)

views目录下的businessDealt（业务办理）qualificatApply（资质申请页面）

页面基础文件文件包含index.html与模块脚本index.js 

index.html为一级页面主视图，视图文件继续包含
```html
<title>{{当前页面标题}}</title>
<span class="layui-breadcrumb" lay-separator="&gt;">
    <a href="#/">{{一级面包屑路径}}</a>
    <a>{{二级面包屑路径}}</a>
    <a>{{当前页面名称}}</a>
</span>
```
作为页面加载时，渲染面包屑使用

### 模板文件中的css
 <font color=green>注：当前没有做一级界面为数据模板，后期可以考虑面包屑以参数的形式，或者在hash处做处理。目前暂定用这种方式</font>

模板文件中的css为当前模板渲染使用到的样式，一般不做过多的样式处理（基本上页面布局都以全局scss文件中的类名作为布局，除非当前模板有些需要特殊处理的。）

如果有涉及到需要做些样式调整，需在模板最外层定义专属模板的class类名 以作权重。如模板：
```html
<span class="layui-breadcrumb" lay-separator="&gt;">
    <a href="#/">首页</a>
    <a>业务办理</a>
    <a>证书变更申请</a>
</span>
<div class="layui-row layui-col-space15 qualification">
  <div class="layui-col-md12">
    <div class="tips">
        <ul class="tips-info">
            <li>撤回申请经主管部门批准后可以重新编辑安全证申请书和相关证明材料；</li>
            <li><span>2、企业安全生产许可证核发办事指南请参考：</li>
            <li>3、未通过省工商局“多证合一”数据查询接口验证的企业，不允许提交安全证申请。</li>
        </ul>
    </div> 
      <!-- 填充内容 -->
      <table class="layui-hide" lay-filter="changeRequest" id="changeRequest"></table> 
  </div>
</div>
```
当前模板我们需要对全局的tips-info做些局部调整，则style样式中，将以qualification作为权重约束
```css
.qualification .tips{
    margin-top:20px
}
```

### 模板文件中js的基本规范
模板中的js与模板文件放在同一级目录底下，方便维护代码

当前模板中的js均为按需加载，由于layui模块的异步加载原因，我们写的模块js脚本必须在所需的组件加载完，再通过seajs引入。
基本规范为：
```js
 <script>
      layui.use(['admin','element', 'form', 'table', 'laydate', 'upload'], function () {
      var $ = layui.$,
        element = layui.element,
        form = layui.form,
        table = layui.table,
        laydate = layui.laydate,
        router = layui.router();
  
      seajs.use('./views/businessDealt/changeRequest/index', function (e) {
        new e()
      })
    })
  </script>
```
其中，本页面的模板中做使用到的layui插件，必须在当前模板js中，通过layui.use加载，因为模块是属于通过路由加载的，有些基础插件<font color=green>类似layer、element、form</font>我们在初始化项目的时候有加载缓存，但当前模板除了基础插件外，都得在单独模板中独立加载。
<font color=red>注意：在开发中，有时候我们中A页面跳转到B界面，假设A界面有使用到tree插件，并且A界面有加载tree。当我们做界面跳转的时，插件是有缓存的，B界面如果没有加载tree 并且有使用到tree,并不影响正常使用。

但当在hrel处在B界面时，刷新浏览器，我们是不会去加载tree的，这样就会影响到B模板的功能正常使用。所以为了不影响正常的插件使用，建议当前模板有使用到的插件，都在当前模板的layui.use中做加载。并不会照成资源重复加载</font>

### 模块脚本的路径引用
在这里，虽然layui本身有提供模板的模块脚本编写方式，但他们之间的依赖关系以及前置加载约束，本项目采用sea.js编写模块脚本，更有利于维护模块脚本之间的依赖关系以及按需加载。

对于sea.use中引用的路径，由于在配置sea.js时，base配置为当前项目的根路径，所以在加载脚本时，不得不由跟路径的方式去查找到当前模板对应的脚本文件并加载。后期有更好的方式解决，随时欢迎PR
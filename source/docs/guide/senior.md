title: 模块中的高级写法
---
在项目开发中，模块的写法，往往是不可能像上文[模块的js编写](/public/docs/guide/first_js.html)那么简单，而且页面经常还包括二级页面（类似弹窗详情、修改之类）。如何做到更好的去管理模块功能之前的代码？我觉得这是门艺术，并不是我所规定的规范。所以，以下只是一中idea。可以借鉴。更多的是靠自己怎么取组织与构建。

### 重点
本篇将重点围绕：模块代码解耦、模块代码复用、模块代码如何提取与分离、模块的桥接，代理模式的运用做介绍

### 模块代码解耦
我们先看个具体的运用场景：
![](https://thumbnail10.baidupcs.com/thumbnail/ebb3202d19f648d078fcedcb660b97cb?fid=1117135785-250528-185447622760946&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-vtjPX2Gu6SGGyAZXViGK9ZzEDu4%3d&expires=8h&chkbd=0&chkv=0&dp-logid=2028414447996841840&dp-callid=0&time=1553828400&size=c1920_u1080&quality=90&vuk=1117135785&ft=image&autopolicy=1)
资质申请界面功能包含 新增申请功能，详情功能等。在没有触发这些操作之前，这部分的视图文件以及执行的逻辑处理脚本，就不应该加载，而是当用户操作这部分功能时，我们在从服务器获取这部分资源渲染与执行，同时做资源缓存，必要时再做释放。
我们从程序的角度来简单的实现这部分的功能。以[模块js编写](/public/docs/guide/first_js.html)的模板为例
```js
define(function (require, exports, module) {
     var MODULE_NAME = "qualificatApply";//模块名称

     function Init(params) {
         var defaults = {
             //...Object
         };
         var _this = this;
         _this.MODULE_NAME = MODULE_NAME;
         _this.defs = $.extend({}, defaults, params);
         _this.render().done(function() {
             _this.toolbarEvent()
         })
     }
     Init.prototype = $.extend({}, new Class(), {
         constructor: Init,
         /**
          *初始化模块入口执行函数
          */
         render:function() {
             var $dfd = $.Deferred()
             //渲染表格..此处先忽略渲染的脚本
             return $.dfd.resolve()//假设表格渲染完，抛出resolve
         },
         /**
          *注册表格的新增与详情事件
          */
        toolBarEvent:function() {
            layui.table.on('toolbar('+MODULE_NAME+')', function (obj) {
                 var toolevent = obj.event;
                 if (toolevent == 'add') {
                     require('./add').call(_this)
                 }
             });
             //表格操作事件注册
             layui.table.on('tool('+MODULE_NAME+')', function (obj) {
                 var toolevent = obj.event;
                 if (toolevent == 'info') {
                     var Info = require('./info');
                    new Info()
                 }
             });
        }
         
     })
     return Init;
 })
```
这里，我做了三个内容的操作
- 1. 渲染表格
- 2. 注册表格的新增与详情操作
- 3. 加载对应操作的js文件

#### 渲染表格
此处暂时先忽略渲染表格的具体内容，表格的渲染方式，可以查看layui文档，本项目中统一封装了表格渲染的模块，这里不做过多讲解，可以查看plugins=>render_table.js模块。

这里的表格渲染函数，项目实现上抽离到父构造函数中做资源加载与统一管理执行 (在businessDealt目录底下有个index.js做公用函数渲染与资源加载) ，在所里这里先简单带过。后续章节在讲解

#### 注册表格事件
```js
toolBarEvent:function() {
    layui.table.on('toolbar('+MODULE_NAME+')', function (obj) {
        var toolevent = obj.event;
        if (toolevent == 'add') {//新增操作事件
            require('./add').call(_this)
        }
    });
    //表格操作事件注册
    layui.table.on('tool('+MODULE_NAME+')', function (obj) {
        var toolevent = obj.event;
        if (toolevent == 'info') {//详情事件
            var Info = require('./info');
            new Info()
        }
    });
}
```
toolbarEvent 注册当前表格所需的事件，并在render函数（渲染表格）加载并执行完毕之后，在注册表格事件。

我们将事件触发所需执行的代码，抽离到新的模块中编写（add.js||info.js）。

可能你也发现，新增操作时，我们使用的都是sea.js的同步加载，但是写法并不一致。
因为，add.js模块的写法，是一个函数，我们只要新增的时候，执行新增的逻辑即可。而info.js是一个构造函数，我们需要在使用的时候，去实例化它（<font color=blue>这里的写法是为了后面做代理模式的讲解做铺垫，此章节不做太多解释</font>）。如果你对性能有所了解，你会察觉到，详情事件模块的写法，会重复实例化构造函数，造成内存不必要的开销，这里我们就需要在此处做一层代理。我们简单的做下改进（此处只针对info.js模块加载并执行的写法）
```js
layui.table.on('tool('+MODULE_NAME+')', function (obj) {
    var toolevent = obj.event;
    if (toolevent == 'info') {//详情事件
        if(_this.infoMod) {
            var Info = require('./info');
            _this.infoMod = new Info()
        }else{
            _this.infoMod()
        }
    }
});
```
### 模块代码复用

前面我们提到渲染表格的具体写法，本项目一级页甚至二级、三级页都很多需要涉及到表格渲染，表格渲染的函数统一封装在table_render.js模块，由于每个项目的接口数据不尽相同，在此模块做统一的数据请求处理，以及修改请求数据与数据表格渲染所需要的参数配置。

业务办理模块的二级页都涉及到表格渲染，那么我们是否可以将表格渲染与表格涉及到的一些操作，统一封装起来，供每个二级页做加载呢？

我们在businessDealt底下建index.js。此模块将当前模块底下所有二级页的通用函数封装起来，例如：每个模块都涉及到页面表格渲染。

所以我们代码这样写：
```js
 /**
  * @author NYG
  * @class 业务办理模块的继承类，用于统一执行业务办理模块公用模块处理
  */
 define(function (require, exports, module) {
     var PTable = require('js/plugins/render_table');

     function Class(params) {
         var defaults = {};
         this.PTable = PTable;
         this.defs = $.extend({}, defaults, params)
     }
     Class.prototype = {
         constructor: Class,
         /**
          *表格初始化渲染
          * @returns
          */
         renderTable: function () {
             var _this = this,
                 defs = _this.defs;
             var dfd = new $.Deferred();
             PTable({
                 id: defs.id,
                 elem: defs.elem,
                 cols: defs.cols,
                 url: defs.url,
                 toolbar: defs.toolbar || '',
                 cbk:function() {
                     return dfd.resolve(arguments)
                 }
             })
             return dfd.promise()
         }
     }
     return Class;
 })
```
因为所有二级页都需要渲染数据表，所以都需要调用render_table.js模块，我们将之抽离到当前业务办理大模块的共用模块做加载（<font color=blue>此函数可以将之理解为vue中局部混入minxins使用</font>）;我们将表格的渲染统一封装在renderTable函数里面，供当前模块二级页调用，函数本身抛出一个promise对象，供回调使用。这里只是做简单的封装处理。具体看使用情况做修改。

#### 语法糖
- 1. 公用业务组件做分离，提取上层做封装，供使用模块做继承执行。
- 2. Deferred函数，这是jquery里面的一种回调处理机制，在项目使用中会替代传统的参数回调。
- 3. 构造函数继承，混合继承


### 模块代码提取与分离
模块的代码的提取与分离是处理好项目中模块写法的一个重要手段，大致提取的原则：

#### 提取的规则
> 如果当前模块（业务办理）中，底下二级菜单都有涉及到的业务场景，例如renderTable,dialog等，将之渲染函数提取到当前模块js（假设我们这边定义为<font color=blue>subA模块</font>）代码中，供其他模块继承复用。
如果项目里面例如与<font color=blue>subA模块</font>属于同级别的模块任有复用代码，我们将之提取到顶级<font color=blue>假设为parent模块</font>，在SubA模块中做继承使用。这部分概念可能会有点抽象，我们以代码的方式举个例子。
<font color=red>（以下代码只做案例讲解，不涉及到真实的业务代码）</font>
假设我们在common有个公用模块代码，Parent.js,代码为：
```js
function ParentClass (params) {
    var defaults = {}
    this.defs = $.extend({},defaults,params)
}
ParentClass.prototype = {
    constructor:ParentClass,
    renderTable:function() {},
    dialog:function() {},
    //...
}
```
业务办理模块的公用模块我们定义为Sub.js，代码为：

```js
var Parent = require('Parent')
function Sub (params) {
    var defaults = {}
    Parent.call(this);
    this.defs = $.extend({},defaults,params)
}
ParentClass.prototype =$.extend({},new Parent,{
    constructor:Sub,
    //...
}) 
```
Sub.js继承Parent模块。同时可以在当前模块中做数据自己模块的一些个性化函数扩展。

资质申请中的js模块，我们定义为child.js,代码为
```js
var Sub = require('Sub')
function Child (params) {
    var defaults = {}
    Sub.call(this);
    this.defs = $.extend({},defaults,params)
}
ParentClass.prototype =$.extend({},new Parent,{
    constructor:Child,
    //...
}) 
```
当我们在实例化Child模块时，我们需要执行renderTable函数，就可以这样使用
```js
var Child = require("child")
var child = new Child();
child.renderTable()
```
公用函数的提取，遵循从<font color=blue>底部往上逐级提取</font>，不可在顶级的构造函数中滥定义没有高度复用的代码，如果只是当前模块中复用度比较高的，只要在当前模块提取即可，类似（Sub.js）

#### 模块的分离
在一个页面中，模块的代码不仅仅只是我们打开所呈现出来的部分，往往还涉及到操作，例如增、删、改、详情之类的操作。那么，我们要做的事情，就是打开当前页面，我们只加载当前呈现出来的部分代码，关于操作部分的代码，我们只有等真正操作的时候，再去加载与渲染。

在当前系统（住建行政审批平台）中，很大部分的页面详情，在系统中都有可能复用到，以往的方式，我们只能将这部分的代码放置单独页面做，以iframe的方式渲染达到复用的效果。

但既然我们定义的是单页，作为有原则程序猿，我们坚持不用这种方式去做复用模块的加载。借用vue公用模块的思想，既然定义是公用，就不该与当前模块耦合性太高，我们将vue中的props以模块的参数作为实现，模块的执行逻辑也在本身模块加载的同时进行加载与执行（当然，我们不会将js与模板写在一起，我们会根据一定的规则，将他们关联起来）同时我们将打开复用模块的入口统一在一个模块(bradge.js)中做管理，（vue的实现也尽量将公用模块的入口，放置状态管理（store）中进行统一管理）
下面，我们看一下具体在项目中的实现。还是以业务办理=>资质申请模块中的详情弹窗为例：
![](https://thumbnail10.baidupcs.com/thumbnail/5479d87570eae060ce49d23bd5a6c61a?fid=1117135785-250528-1077374451345446&rt=pr&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-DPM%2fjc337FdjLupcGLSVWhOtO%2f0%3d&expires=8h&chkbd=0&chkv=0&dp-logid=2074212011150021843&dp-callid=0&time=1553997600&size=c1920_u1080&quality=90&vuk=1117135785&ft=image&autopolicy=1)

当前页面模块中，我们引入了她的上层index.js（这里可以理解为上文提到过的Sub.js）同时赋值给当前模块Class变量。
随之，我们在当前模块的构造函数中做继承
```js
Class.call(this);
```

以及原型扩展

```js
Init.prototype = $.extend({},new Class,{//...})
```
重点在于，我们是如何将详情功能分离出去的，这里执行的_this.info()是在上层index.js中定义的，对于特殊处理，我们以参数的形式传递来控制info做不同详情的渲染加载。

info函数做了什么事情呢？
我们看项目中的代码
```js
/**
 *
 *详情操作
* @param {*} params 
* params.BgFn 参数用于控制具体模块执行时需要调用bridge里面的那个详情方法
* params.base_page 详情页面首次渲染的视图文件名
*/
info: function (params) {
    var _this = this,
            defs = _this.defs;
    var $dfd = new $.Deferred();
    try {
        bridge[params.BgFn](params.data)
        $dfd.resolve()
    } catch (error) {
        $dfd.reject(error)
    }
    return $dfd.promise()   
}
```
这里你会发现，info函数又做了一层封装，参数用于控制从bradge中调用执行哪个函数，以及做参数传递使用。为什么要这么麻烦呢？其实并不然。

上文有提到过，由于本项目中，大部分详情，都有可能存在复用的情况。那么，我们就不能跟当前模块的业务做绑定太多。我只要知道当前模块中的详情，我需要调用哪个函数，传递什么参数过去就行。如上述代码中：
```js
_this.info({
    BgFn:'renderInfo_QlfcationsApply',
    data:{
    BASE_PAGE:'info_csbj'
    }
});
```
我们只要关心，当前模块的详情，需要调用bridge中的哪个函数名称，以及初始化渲染的模块名称即可（<font color=green>因为从不同的入口进入同一个详情页，有可能初始化时候，加载显示的模板不同，所以需要用参数控制</font>）

这样极大的方便不是开发这个模块的开发者调用，同时也极大的方便自身维护这些公用的模块。

好了，在理解了上述的操作之后，你可能会发现，这些模块是如何渲染模板以及加载模板对应的脚本的呢？[bridge中的函数](/public/docs/guide/senior.html#bridge中的函数)

#### bridge中的函数
我们以上文info函数中参数 bgFn:'renderInfo_QlfcationsApply'为例：代码为
bridge.js
```js
/**
 *渲染资质申请详情弹窗模块
    *@param {Object} params
    *params.MODULE_NAME表示模块名称
    *params.id 模块请求数据的ID,
    *params.BASE_PAGE:当前详情页首次加载需要加载的模板名称
    */
renderInfo_QlfcationsApply: function (params) {
    var _this = this,
        defs = _this.defs;
    _this.showInfoDialog({
        NAME: defs.QANAME,
        COMP: defs.QACOMP,
        BASE_PAGE: params.BASE_PAGE || 'info_csbj',
        URL: './json/tree.json'
    })
},
```
NAME为当前模块的名称，在[模块js编写](/public/docs/guide/first_js.html#模块js规范)中我们有提到，NAME即为当前模块的一个全局名称，用于事件句柄定义与模块路径的规则定义。
COMP为所需加载模板的路径
BASE_PAGE即为初次渲染加载的模板名称，这是对外暴露的参数，供调用的时候定义的。
URL为渲染详情左侧树的请求地址

我们再看_this.showInfoDialog做了哪些处理
bridge.js
```js
/**
 *@global 详情公用弹窗函数，不允许私自修改 
    *@author NYG
    * @param {*} params 参数包含模块的名称params.NAME 以及模块路径 params.COMP
    * params.BASE_PAGE 表示当前详情页如果有树形列表，需要加载的第一个模板名称
    */
showInfoDialog: function (params) {
    var _this = this,
        defs = _this.defs;
    ctrl.renderDialog({
        id: params.NAME + 'Info'
    })
    .done(function () {
        // 加载详情视图文件
        layui.view(params.NAME + 'Info')
        .render(params.COMP + 'info')
        .done(function (tpl) {
            //渲染左侧菜单视图文件
            if (params.URL) {
                _this.renderInFoTree({
                    NAME: params.NAME,
                    COMP: params.COMP,
                    PATH: params.COMP.substring(3),
                    URL: params.URL
                })
                //加载参数 BASE_PAGE视图文件 作为首次打开详情的页面
                layui
                    .view(params.NAME + '_cont')
                    .render(params.COMP + 'html/' + params.BASE_PAGE)
                    .done(function () {
                        //首次加载的模块脚本
                        _this.loadModuleJs({
                            PATH: params.COMP.substring(3),
                            alias: params.BASE_PAGE
                        })
                    });
            }
            layui.form.render();
        });
    })
    .fail(function () {});
},
```
首先，我们调用了ctrl中的公用弹窗函数，其次，我们在他的异步回调函数中，做了渲染当前详情模板的加载：
```js
layui.view(params.NAME + 'Info')
    .render(params.COMP + 'info')
    .done(function (tpl) {//...})
```
params.NAME + 'Info'为模板的容器ID,params.COMP + 'info'为容器加载的模板地址，.done()函数我们作为加载完基础模板之后做的异步回调处理

在.done()中，我们做了一件事：如果当前视图模板有树形菜单，根据params.URL判断是否需要渲染树，如果存在，调用
```js
_this.renderInfoTree()
```
同时加载参数中传递进来的模板名称对应模板，并加载与之对应的模板脚本。

renderInfoTree中，我们根据URL渲染树形菜单，同时注册点击事件
```js
/**
 *@global 详情公用创建树形菜单函数，不允许私自修改
    *@author NYG
    * @param {*} params 参数包含所创建树的模块所需加载的js路径，params.PATH
    * params.NAME 当前模块名称，用于创建树形菜单
    * params.URL 准备后续添加，用于传入请求接口
    */
renderInFoTree: function (params) {
    var _this = this,
        defs = _this.defs;
    new MODULE_TREE({
        renderDom: '#' + params.NAME + '_tree',
        url: params.URL,
        click: function (node) {
            if (!node.alias) return;
            //加载对应模块与脚本
            layui
                .view(params.NAME + '_cont')
                .render(params.COMP + 'html/' + node.alias)
                .done(function () {
                    _this.loadModuleJs({
                        PATH: params.PATH,
                        alias: node.alias
                    })
                });
        }
    });
},
```
在点击事件中，我们根据当前点击的节点，获取到对应的数据，加载模板与执行加载模板js（<font color=green>_this.loadModuleJs()</font>）。

这样，我们就实现了一套完整的分离流程，同时保证所有复用的模块在bridge中做统一的管理与维护。在实际页面模块中，我们只需传递几个参数就能实现公用模块的调用。
### 模块的依赖处理
...待更新
### 代理模式运用
...待更新
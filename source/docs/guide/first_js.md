title: 路由视图的js编写
---
### 规范
模块代码遵循seajs规范，具体查看<font color=green>js规范</font>，需要特别申明，如果模块是构造函数写法，尽量在模块代码中使用代理模式编程，防止构造函数重复的实例化占用内存，照成没必要的性能开支。

下面以 业务办理==>资质申请模块的js为例：
### 模块js规范

创建模板，可以将此代码作为所有模块js的模板进行编写
```js
 /**
  * @author NYG
  * @class 资质申请模块,用于初始化
  */
 define(function (require, exports, module) {
     var MODULE_NAME = "qualificatApply";//模块名称

     function Init(params) {
         var defaults = {
             elem: '#'+MODULE_NAME,
             id: MODULE_NAME,
             url: "./json/test.json"//当前模块的请求地址
         };
         var _this = this;
         _this.MODULE_NAME = MODULE_NAME;
         _this.defs = $.extend({}, defaults, params);

         _this.render().done(function () {

         });
     }
     Init.prototype = $.extend({}, new Class(), {
         constructor: Init,
         /**
          *初始化模块入口执行函数
          */
         render:function() {
             var _this = _this,defs = _this.defs;
             return _this;//做链式调用使用，这里不做限制，也可以使用deferred控制异步回调
         }
         
     })
     return Init;
 })
 ```
 在代码中，约定模块名称（<font color=red>MODULE_NAME</font>）作为参数，用于做<font color=red>事件注册</font>以及<font color=red>模板加载</font>的一个核心依据，页面上，凡是涉及到需要操作dom的地方，均已模块名称+"_操作名"的形式定义。例如页面代码中表格渲染的id以及lay-filter的命名 

 ```html
 <table class="layui-hide" lay-filter="qualificatApply" id="qualificatApply"></table>
 ```

 例如界面有查询功能，则表单的lay-filter则命名为qualificatApply_search
 ### Init构造函数

 模块构造函数包含三部分

 #### 初始化传参
构造函数Init的唯一形参params用于外部调用当前模块时，实例化传的参数，构造函数本身会默认做些初始化参数配置:
```js
var defaults = {}
```
当我们需要修改默认参数，或者是做些模块特殊处理时，将传进来的参数params与defaults合并，赋值给当前对象的defs属性。

```js
_this.defs = $.extend({}, defaults, params);
```

#### 执行初始化函数render
原型链上的render函数，用于初始化当前模块的脚本，在这里，我们可以做表格渲染，表单验证，以及各种事件的注册。默认情况下，在实例化当前构造函数时，就执行:
```js
_this.render().done(function () {

});
```

好了，一个基础的js模板完成。
这里你可能会觉得.done()是什么？这是函数中处理异步回调的写法，更多写法，请转[模块的高阶写法](/public/docs/guide/senior.html)。
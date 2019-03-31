title: 语言规范
---

## 语言规范

项目js统一使用commonJs规范，由于本项目大部分模块脚本都是以构造函数的方式编写，构造函数继承与原型继承的方式，本章节将重点介绍项目中js的构建体系。
项目中主要采用seajs作为模块依赖与加载，在开始开发之前，有必要简单介绍下seajs文档的基础配置

### 快速简要的知识点
- seajs.config({...}) 用来对seajs配置
- seajs.use([a,b],function(a,b) {...})用来加载一个或者多个js模块
- define(function(require, exports, module){...}) 用来定义模块，sea.js推崇一个模块一个文件，遵循统一的写法
- require('xxxModule') 用来获取指定模块的接口
- require.async 用来在模块内部异步加载一个或者多个模块
例如：
```js
define(function(require){
    require.async(['aModule','bModule'],function(a,b){  // 异步加载多个模块，在加载完成时，执行回调
    a.func();
    b.func();
    })    
});
```
- exports, //用来在模块内部对外提供接口。 例如：
```js
define(function(require, exports){
    exports.varName01 = 'varValue';  // 对外提供 varName01 属性    
    exports.funName01 = function(p1,p2){  // 对外提供 funName01 方法
    ....
    }       
});
```
- module.exports, 与 exports 类似，用来在模块内部对外提供接口。例如：
```js
define(function(require, exports, module) {  
  module.exports = {  // 对外提供接口
    name: 'a',
    doSomething: function() {...};
  };
});
```
以上为seajs最常用的，必须牢记于心。
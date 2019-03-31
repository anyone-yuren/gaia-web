title: 开始编写
---

## 编码规范

统一团队的编码规范，有助于代码的维护。本章是传统意义上的 `Style Guideline`，目的是统一一些相对主观化的代码风格。

那么，重点不在于一些常规的规范，如果需要了解常规的规范，请转[慧政通-js代码规范](http://192.168.169.57/ue/code/public/docs/js/code.html),本文将重点讲解项目中模块脚本的编写风格与基础模板


一个简单的模块代码在本项目中应该包含哪些呢：
### 函数型基础模板
```js
define(function(require,exports,module){
    /**
     * 基础函数写法
     * @params params用于执行实例化所传递的具体参数
     **/
    function init(params) {
        var _this = this;//控制作用域不被篡改
        //...执行模块代码
        renderTable()
    }
    /**
     * 注意，由于seajs模块本身就是个单例，我们不想对外暴露的，或者私有属性和方法
     * 统一使用_+函数名的方式定义
     **/
    function _renderTable() {}
    
    return init;//等同于exports.init = init
})
```

### 构造函数基础型模板
```js
define(function(require,exports,module){
    /**
     * 构造器
     * @params params用于执行实例化所传递的具体参数
     **/
    function Class(params) {
        var defaults = {}//用于配置当前模块的基础参数
        var _this = this;
        _this.defs = $.extend({},defaults,params)
    }
    //原型链扩展
    Class.prototype = {
        constructor:Class,
        render:function() {//...}
    }
    return Class;//等同于exports.Class = Class
})
```
### 构造函数继承型模板
```js
define(function(require,exports,module){

    var ParentClass = require("../parentClass");
    /**
     * 构造器
     * @params params用于执行实例化所传递的具体参数
     **/
    function Class(params) {
        var defaults = {}//用于配置当前模块的基础参数
        var _this = this;
        ParentClass.call(_this);//函数式继承
        _this.defs = $.extend({},defaults,params)
    }
    //原型链继承扩展
    Class.prototype = $.extend({},new ParentClass(),{
        constructor:Class,
        thisRender:function() {}
    })
    return Class;//等同于exports.Class = Class
})
```
其中构造函数继承型模板在项目中最常用。具体还是以业务交互场景来定。不做特殊约束。
title: 起步
---
## js中的继承
在起步之前，我觉得蛮有必要再次介绍下js中如何实现继承.简单的继承关系，这里只简单待过，不做过多说明，如需了解，可查阅[ECMAScript 继承机制实例](http://www.w3school.com.cn/js/pro_js_inheritance_in_action.asp)

### 构造函数继承

使用call和apply借用其他构造函数的成员，可以解决给父构造函数传递参数的问题，以及共用私有属性和方法。但是获取不到父类构造函数原型上的成员，也不存在共享问题。
```js
//创建父构造函数
function Parent (name) {
    this.name = name;
    this.say = function() {
        console.log(this.name)
    }
}
//创建子类构造函数
function Sub(name) {
    Parent.call(this,name)
}
//测试是否有Parent的成员
var sub = new Sub('sunada');
sub.say();// sunada
```
### 原型链继承
所有的实例对象，有一个内部的指针，指向它的原型对象，并且可以访问原型对象上的属性和方法，那么我们在创建构造函数时，这样我们就可以在当前构造函数的原型上继承需要被继承的构造函数实例。举个例子

```js
//创建父构造函数
function Parent() {

}
//创建Parent原型对象
Parent.prototype = {
    constructor:Parent,
    say:function() {}
}
//创建子构造函数
function Sub() {

}
Sub.prototype = new Parent();
Sub.prototype.constructor = Sub
```
将Sub的原型进行扩展，同时，将Sub的指针指向Parent的原型对象。当我们实例化Sub构造函数时，实例化对象既能访问Sub的原型对象，同时也能访问Parent的原型对象

### 组合继承

构造函数继承+原型链继承
```js
//创建父构造函数
function Parent() {

}
//创建Parent原型对象
Parent.prototype = {
    constructor:Parent,
    say:function() {}
}
//创建子构造函数
function Sub() {
    parent.call(this)
}
Sub.prototype = new Parent();
Sub.prototype.constructor = Sub
```
但是这种方式会存在共享问题，在实例化对个Sub时，如果这些后代对象上修改了原型，会影响到处在同一共享原型链上的所有对象，这里有很多种方式可以避开这样的编码漏洞。下面介绍下项目中常用的方式。

### 原型继承深拷贝
由于项目是基于jquery搭建，我们可以借助$.extend函数进行深拷贝(也可以使用lodash.js)
```js
function Parent(name) {
    this.name = name
}
Parent.prototype = {
    constructor:Parent,
    say:function() {
        console.log(this.name)
    }
}

function Sub () {
    Parent.call(this)
}
Sub.prototype = $.extend({},new Parent(),{
    constructor:Sub
})
```
之前我写的文档有关于原型链模式的介绍与讲解，有兴趣可以前往[大话原型链模式](http://note.youdao.com/noteshare?id=102a915c7df2db1598d545bc48589638)
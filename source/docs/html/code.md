title: 代码规范
---

### 团队约定

HTML文件必须加上 DOCTYPE 声明，并统一使用 HTML5 的文档声明：

```html	
	<!DOCTYPE html>
```
### 模板规范

项目中模板机制采用的是layui.tpl规范，严格按照[layui官方文档](https://www.layui.com/doc/modules/laytpl.html)规范进行编写

项目中的模板统一采用的view模块中的render函数进行渲染，具体实例可参考文档[admin view模块](http://192.168.169.57/ue/projects/admins/document/docs.html#base-view) 项目中使用，可参考bridge.js模块中的`showInfoDialog`函数写法

### 模板模块中ID规范
凡涉及到操作与模板容器的地方，统一使用id进行标识。由于项目构建采用的是单页，严格避免模块中出现重复的ID，以免路由切换时造成事件的重复渲染。<font color=red>id的命名规范，使用驼峰式写法</font>。例如<业务办理=>资质申请模块> 一级页表格id名称为

```html
qualificatApply
```

详情弹窗中的树形菜单id为

```html
qualificatApply_tree
```

详情弹窗中右侧容器的id名为

```html
qualificatApply__cont
```

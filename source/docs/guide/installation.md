title: 安装
---
>  项目采用gulp作为工程搭建，包含模板编译（gulp-n-includer），scss编译（node-scss） js合并压缩、html压缩、等常规操作。本项目采用git做版本管理。所以，确保系统安装nodejs与git
<font color='red'>建议node版本安装6.9.2 其余项目（vue）使用nvm做node版本管理
### 兼容性
兼容IE8,所使用的语法与模块依赖采用sea.js做管理。路由监听hash变化。项目要求：兼容IE9

### 更新日志
目前第一版：<font color="green">v0.0.1</font>
版本更新迭代，查看git日志

### clone项目

```
 git地址： http://192.168.169.57:9000/scm/git/gaia-web.git
 ```
 git账户名与svn账户名一致 例如(账号:nyg 密码:hzt_123456)

### 安装依赖
建议使用cnpm 安装，由于部分插件的兼容问题，不建议使用yarn安装；
```
cnpm install
```
### 运行与打包
项目目前只做了task wacth与default任务，开发环境与生产环境配置更新中...，目前启动项目运行与打包统一执行
```
gulp
```
# 踩坑记录
## 下载下来之后执行npm install安装依赖(保证安装了node.js和npm),过程中假如node-sass安装错误。执行以下步骤:
①：npm rebuild node-sass
②：npm uninstall node-sass
③：npm config set registry http://registry.npm.taobao.org/
④：npm i node-sass --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
执行完之后可以尝试运行npm run dev
## 编译报错或者页面的Console总是会出现莫名的错误
书写代码完毕之后运行，编译期间的控制台或者页面的Console总是会出现莫名的错误，貌似是ESLint的语法检测。
可以在idea的快捷键设置中搜索esl关键字，打开Fix ESLint Problem快捷键并设置1个常用的快捷键，例如ctrl+alt+小键盘的1,然后就只需要在书写完成之后使用快捷键进行整理即可。



## renren-fast-vue
- renren-fast-vue基于vue、element-ui构建开发，实现[renren-fast](https://gitee.com/renrenio/renren-fast)后台管理前端功能，提供一套更优的前端解决方案
- 前后端分离，通过token进行数据交互，可独立部署
- 主题定制，通过scss变量统一一站式定制
- 动态菜单，通过菜单管理统一管理访问路由
- 数据切换，通过mock配置对接口数据／mock模拟数据进行切换
- 发布时，可动态配置CDN静态资源／切换新旧版本
- 演示地址：[http://demo.open.renren.io/renren-fast](http://demo.open.renren.io/renren-fast) (账号密码：admin/admin)

![输入图片说明](https://images.gitee.com/uploads/images/2019/0305/133529_ff15f192_63154.png "01.png")
![输入图片说明](https://images.gitee.com/uploads/images/2019/0305/133537_7a1b2d85_63154.png "02.png")


## 说明文档
项目开发、部署等说明都在[wiki](https://github.com/renrenio/renren-fast-vue/wiki)中。


## 更新日志
每个版本的详细更改都记录在[release notes](https://github.com/renrenio/renren-fast-vue/releases)中。

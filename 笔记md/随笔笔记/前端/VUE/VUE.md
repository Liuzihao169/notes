> 相关文档：
>
> https://cn.vuejs.org/v2/guide/
>
> 模板ui：vue-element-admin
>
> https://www.zhihu.com/question/24399966
>
> vue+iview后台管理模板
>
> https://github.com/iview/iview-admin
>
> 模版推荐
>
> https://www.jianshu.com/p/0f41bfe211a8
>
> ant design
>
> vue-admin-template 模版



#### 一、什么是VEU

> 是一套构建用户界面的渐进式框架。Vue 只关注视图层， 采用自底向上增量开发的设计。

它实现了MVVM架构。 M:数据 V:视图 VM:是我们的view对象。view对象帮我们实现，数据绑定和监听的工能；

#### 二、VUE的生命周期

<img src="https://gitee.com/liuzihao169/pic/raw/master/image/lifecycle.png" alt="The Vue Instance Lifecycle" style="zoom:50%;" />

#### 三、VUE如何打包部署

1.打包部署到server上

2.打包部署到tomcat服务器上

> a.在web pack.prod.config.js 文件中添加暴露的。项目名 配置

```java
publicPath : '/dist文件名/'
```

> b. 使用 npm run bulid 打包生成 dist 文件

> c.修改成与 暴露项目名后，放到tomcat中的 root文件夹中

#### 四、组件化编码

1.拆分  2.静态组件  3.动态组件

#### 五、路由

1、编写路由组建

2、配置路由器，

3、配置路由

4、使用 router-link 、router-view




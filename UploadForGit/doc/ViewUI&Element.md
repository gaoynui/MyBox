# View UI学习

## 安装

使用npm:

```
npm install view-design --save
#查看版本
vue -V
```

或<script>全局引用:

```
<script src="//unpkg.com/view-design/dist/iview.min.js"></script>
```

其他安装vue.js和stylesheet:

```
<!-- import Vue.js -->
<script src="//vuejs.org/js/vue.min.js"></script>
<!-- import stylesheet -->
<link rel="stylesheet" href="//unpkg.com/view-design/dist/styles/iview.css">
```

## 单文件组件

```
<template>
	// 页面设计
</template>
<script>
	// 页内逻辑
</script>
<stype>
	// 样式设计
<stype>
```

## ElementUI

安装

```
npm i element-ui -S
```

或者引入CDN资源

```
<!-- 引入样式 -->
<link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
<!-- 引入组件库 -->
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
或者在main.js中引入：
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
注：以上两个依赖不引用的话element组件无法正常使用
```

使用：

[官方组件文档](https://cloud.tencent.com/developer/chapter/18052)


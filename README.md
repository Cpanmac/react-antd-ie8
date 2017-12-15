# react-ant-ie8
这是一个以 React, React Router DOM, Ant Design 等现代流行框架写的兼容 IE8 的 demo 栗子.

## 技术参考 和 代码介绍
* 学习并参考了很多[砖家大神](https://github.com/brickspert)的 [react-family-ie8](https://github.com/brickspert/react-family-ie8)项目配置,当然也做了许多改动
	* 增加了一些常用的关于 redux 的 dependencies, 不过小项目和新人用 redux, 感觉属于吃力不讨好的事情, 记得一句话 `如果你不知道要不要用 redux, 那么就不用`, 而且一个用 redux 写的项目想去改动, 代码耦合太高, 改动起来十分费劲, 就并未在代码中使用
	* 如果单纯为了组件之间通信, 一般有如下做法
		* 组件不要嵌套太深, 嵌套三层就算深了
		* 通过父组件的箭头函数进行子组件之间的通信
		* 如果想要两个不关联的组件进行通信, 推荐两个插件, 都简单易用(支持 ie8+)
			* [pubsub-js](https://www.npmjs.com/package/pubsub-js) 上面就有具体栗子
			* [signals](https://www.npmjs.com/package/signals) [栗子](https://github.com/millermedeiros/js-signals/wiki/Examples) 据说 facebook 就是用的这个
	* webpack 配置增加 less 支持
	* webpack 配置增加 ant design 系列的支持
		* 引入 [antd 1.x](http://1x.ant.design) 本想用最新的, 但是 `2.x 不支持 IE8`
		* 引入 [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import), 并更改 `.babelrc`
	* webpack.common.config.js 中 output 绝对路径改相对路径
	* webpack-dev-server 发现一个坑, IE8 不支持 `inline:true`
* package.json 里面 version 更新到对应小版本最新
* cdn 源两个 [cdnjs](https://cdnjs.com) [bootcdn](http://www.bootcdn.cn/)
	* [cdnjs](https://cdnjs.com) 非常全,更新迅速及时,但是国内访问非常慢
	* [bootcdn](http://www.bootcdn.cn/) 不是很多,少量可能更新不及时,不是最新
	* 吐槽一下 "match-media" 两个cdn都没有,但是没有的话 `ant design` 又报错.于是网上找了好久,没找到可靠的,后在 https://www.npmjs.com 里找到,于是通过加入 dependencies, 然后在 index.js 中 import (不知道有没有更好的解决办法)
* dependencies 查询和资料参考来源 [官网](https://www.npmjs.com)
* 使用 copy-webpack-plugin 直接拷贝静态资源
* 移除了 react-hot-loader 在IE中支持不是很好
* 优化 bundle-loader 组件的创造函数 见`src/utils/bundle.js`
* 对于 ant design 的表格 `表头和列同时固定的时候` 报错 `IE8 不支持 onScroll 事件`, 利用 jquery 做了如下兼容处理
```js
const verIE = e => {
	// 此方法只能检查IE版本 IE11以下 或 IE文档模式11以下
	if ( /*@cc_on !@*/ false) {
		const ver =  /*@cc_on @_jscript_version@*/ -0;
		const mod = document.documentMode;
		return { ver, mod };
	}
};
// 在使用Table的组件中添加如下函数
componentDidMount() {
	const res = verIE(); // 获取IE版本信息,返回值 {ver:IE版本,mod:文档模式版本}
	if (res) {
		if (res.ver < 9 || res.mod < 9) {
			$('.ant-table-scroll .ant-table-body')
				.on('scroll', e => {
					const ev = e || window.event;
					const el = $(e.target || e.srcElement);
					const left = el.scrollLeft();
					el.siblings('.ant-table-header').scrollLeft(left);
				});
			$('.ant-table-fixed-left .ant-table-body-inner,.ant-table-scroll .ant-table-body,.ant-table-fixed-right .ant-table-body-inner')
				.on('scroll', e => {
					const ev = e || window.event;
					const el = $(e.target || e.srcElement);
					const top = el.scrollTop();
					const table = el.parents('.ant-table').eq(0);
					table.find('.ant-table-fixed-left .ant-table-body-inner').scrollTop(top);
					table.find('.ant-table-scroll .ant-table-body').scrollTop(top);
					table.find('.ant-table-fixed-right .ant-table-body-inner').scrollTop(top);
				});
		}
	}
}
```

开始学 react webpack,还有很多不懂,欢迎指点秘籍,或者纠错改进
`fork` 不 `fork` 无所谓, 共同学习,共同进步

## 开发坏境启动
1. `npm install`
2. `npm start`
3. 浏览器打开[http://localhost:8888](http://localhost:8888)

## 生产坏境部署
1. `npm install` 若在前面运行过此命令, 可跳过
2. `npm run app`
3. 拷贝dist文件夹内容至服务器即可

### `重要说明`
前面说到的 proxy 转发设置,一定要在 `开发坏境` `开发坏境` 下才生效,生产环境转发设置就没用了,所以请求会报错的
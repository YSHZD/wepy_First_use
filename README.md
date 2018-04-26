安装 wepy 命令行工具。
npm install wepy-cli -g

wepy new mywepy ( 1.7.0之后的版本使用 wepy init standard mywepy 初始化项目)

开发实时编译。

wepy build --watch


使用微信开发者工具新建项目，本地开发选择dist目录。
微信开发者工具 --> 项目 --> 关闭ES6转ES5。   <!-- 不然会报错 -->
本地项目根目录运行wepy build --watch，开启实时编译。

注意点：

变量与方法使用尽量使用驼峰式命名，避免使用$开头。以$开头的方法或者属性为框架内建方法或者属性，可以被使用，使用前请[参考API文档]()。
入口，页面，组件的命名后缀为.wpy。外链的文件可以是其它后缀。请参考wpy文件说明
使用ES6语法开发。框架在ES6下开发，因此也需要使用ES6开发小程序，ES6中有大量的语法糖可以让我们的代码更加简洁高效。
使用Promise框架默认对小程序提供的API全都进行了 Promise 处理，甚至可以直接使用async/await等新特性进行开发。


vue-cil类似

this.$parent 父组件

以通过修改.wepyrc配置文件，配置自己熟悉的babel环境进行开发。默认开启使用了一些新的特性如promise，async/await等等

程序入口：
入口app.wpy继承自wepy.app，包含一个config属性和其全局属性、方法、事件。其中config属性对应原有的app.json，编译时会根据config生成app.json文件，如果需要修改config中的内容，请使用系统提供API。

标签	    type默认值	     type支持值
style	     css	         css，less，sass（待完成）
template	  wxml	         wxml，xml，html（待完成）
script	      js	         js，TypeScript(待完成)


标签	      lang默认值	    lang支持值
style	        css	              css、less、scss、stylus、postcss
template	    wxml	          wxml、xml、pug(原jade)
script	        babel	          babel、TypeScript


<style type="less">
/** less **/
</style>
<script>
import wepy from 'wepy';
export default class extends wepy.app {
    config = {
            "pages":[
            "pages/index/index"
        ],
        "window":{
            "backgroundTextStyle": "light",
            "navigationBarBackgroundColor": "#fff",
            "navigationBarTitleText": "WeChat",
            "navigationBarTextStyle": "black"
        }
    };
    onLaunch() {
        console.log(this);
    }
}
</script>

单页面：
index.wpy:

<style type="less">
/** less **/
</style>
<template type="wxml">
    <view>
    </view>
    <component id="counter1" path="counter"></component>
</template>
<script>
import wepy form 'wepy';
import Counter from '../components/counter';
export default class Index extends wepy.page {

    config = {};//相当于原来的index.json
    components = {counter1: Counter};//页面引入的组件列表

    data = {};//页面需要渲染的数据
    methods = {};//wmxl的事件捕捉，如bindtap，bindchange

    events = {};//组件之间通过broadcast，emit传递的事件
    onLoad() {};//小程序事件以及其它自定义方法与属性
}
</script>

page =>{
	 
	import wepy from 'wepy';
    import name from '../components/name';
	export default class Index extends wepy.page {

	    data = {
	        
	    };
        
        components = {
	       name:name
	    }; 

	     config = {
	        "navigationBarTitleText": "xxx"
	    };

	    methods = {
	        
	    };

	    onLoad() {
	       
	    };
	}

}

组件： 页面入口继承自wepy.component，属性与页面属性一样，除了不需要config以及页面特有的一些小程序事件等等。
wepy让小程序支持组件化开发，组件的所有业务与功能在组件本身实现，组件与组件之间彼此隔离


com.wpy

<style type="less">
/** less **/
</style>
<template type="wxml">
    <view>  </view>
</template>
<script>
import wepy form 'wepy';
export default class Com extends wepy.component {

    components = {};

    data = {};
    methods = {};

    events = {};
}
</script>


一个.wpy文件分为三个部分：

样式<style></style>对应原有wxss。
模板<template></template>对应原有wxml。
代码<script></script>对应原有js。

其中入口文件app.wpy不需要template，所以编译时会被忽略。这三个标签都支持type和src属性，type决定了其代码编译过程，src决定是否外联代码，存在src属性且有效时，忽略内联代码，示例如下
<style type="less" src="page1.less"></style>
<template type="wxml" src="page1.wxml"></template>
<script>
    // some code
</script>


对现在API进行promise处理，同时修复一些现有API的缺陷，比如：wx.request并发问题等。原有代码：

onLoad = function () {
    var self = this;
    wx.login({
        success: function (data) {
            wx.getUserInfo({
                success: function (userinfo) {
                    self.setData({userInfo: userinfo});
                }
            });
        }
    });
}

async onLoad() {
    await wx.login();
    this.userInfo = await wx.getUserInfo();
}


组件通信与交互：

wepy.component基类提供三个方法$broadcast，$emit，$invoke，
因此任一页面或任一组件都可以调用上述三种方法实现通信与交互
组件的事件监听需要写在events属性下




$broadcast事件是由父组件发起，所有子组件都会收到此广播事件，除非事件被手动取消。事件广播的顺序为广度优先搜索顺序

$emit与$broadcast正好相反，事件发起组件的父组件会依次接收到$emit事件

$invoke是一个组件对另一个组件的直接调用，通过传入的组件路径找到相应组件，然后再调用其方法。


属性	     类型	      默认值	      说明
name	     String     	 -	         事件名称
source	     wepy.component	  -	         事件来源
type	      String	      -	        emit 或者 broadcast
方法	      参数	       返回值	       说明
destroy	       -	        -	          在 emit 或者 broadcast 过程中，调用destroy方法将会停止事件传播。
如果想在Page_Index中调用组件A的某个方法：
this.$invoke('ComA', 'someMethod', 'someArgs');
如果想在组件A中调用组件G的某个方法：
this.$invoke('./../ComB/ComG', 'someMethod', 'someArgs');


$this.$emit('some-event', 1, 2, 3, 4);
events = {
        'some-event': ($event, ...args) {
               console.log(`${this.name} receive ${$event.name} from ${$event.source.name}`);
        }
    };

wepy 数据绑定

wepy使用脏数据检查对setData进行封装，在函数运行周期结束时执行脏数据检查，一来可以不用关心页面多次setData是否会有性能上的问题，二来可以更加简洁去修改数据实现绑定，不用重复去写setData方法。代码如下：

this.title = 'this is title';

<!--  -->
但需注意，在函数运行周期之外的函数里去修改数据需要手动调用$apply方法。如：
setTimeout(() => {
    this.title = 'this is title';
    this.$apply();
}, 3000);

其它优化细节

1. 请求优化

// 官方
wx.request({
    url: 'xxx',
    success: function (data) {
        console.log(data);
    }
});

// wepy 使用方式
// request 接口从只接收Object变为可接收String
wx.request('xxxx').then((d) => console.log(d));

2。事件传参数优化

// 官方
<view id="tapTest" data-hi="WeChat" bindtap="tapName"> Click me! </view>
Page({
  tapName: function(event) {
    console.log(event.currentTarget.hi)// output: WeChat
  }
});

// wepy 建议传参方式
<view id="tapTest" data-wepy-params="1-wepy-something" bindtap="tapName"> Click me! </view>

// 原生的事件传参方式:

<view data-id="{{index}}" data-title="wepy" data-other="otherparams" bindtap="tapName"> Click me! </view>

Page({
    tapName: function (event) {
        console.log(event.currentTarget.dataset.id)// output: 1
        console.log(event.currentTarget.dataset.title)// output: wepy
        console.log(event.currentTarget.dataset.other)// output: otherparams
    }
});

// WePY 1.1.8以后的版本，只允许传string。

<view @tap="tapName({{index}}, 'wepy', 'otherparams')"> Click me! </view>


events: {
    tapName (event, id, title, other) {
        console.log(id, title, other)// output: 1, wepy, something
    }
}

3  改变数据绑定方式

// 官方
<view> {{ message }} </view>

onLoad: function () {
    this.setData({message: 'hello world'});
}


// wepy
<view> {{ message }} </view>

onLoad () {
    this.message = 'hello world';
}

4 组件代替模板和模块

// 官方
<!-- item.wxml -->
<template name="item">
  <text>{{text}}</text>
</template>

<!-- index.wxml -->
<import src="item.wxml"/>
<template is="item" data="{{text: 'forbar'}}"/>

<!-- index.js -->
var item = require('item.js')




// wepy
<!-- /components/item.wpy -->
 <text>{{text}}</text>

<!-- index.wpy -->
<template>
    <component id="item"></component>
</template>
<script>
    import wepy from 'wepy';
    import Item from '../components/item';
    export default class Index extends wepy.page {
        components = { Item }
    }
</script>




当需要循环渲染WePY组件时(类似于通过wx:for循环渲染原生的wxml标签)，必须使用WePY定义的辅助标签<repeat>，代码如下：

/**
project
└── src
    ├── components
    |   └── child.wpy
    ├── pages
    |   ├── index.wpy    index 页面配置、结构、样式、逻辑
    |   └── log.wpy      log 页面配置、结构、样式、逻辑
    └──app.wpy           小程序配置项（全局样式配置、声明钩子等）
**/

// index.wpy

<template>
    <!-- 注意，使用for属性，而不是使用wx:for属性 -->
    <repeat for="{{list}}" key="index" index="index" item="item">
        <!-- 插入<script>脚本部分所声明的child组件，同时传入item -->
        <child :item="item"></child>
    </repeat>
</template>



wepy.component

属性	类型	     默认值	    说明
$root	wepy.page	   -	根组件，一般都是页面
$parent	wepy.component	-	父组件
$wxpage	Page	        -	小程序Page对象
$coms	List(wepy.component)	{}	子组件列表

方法	参数	返回值	    说明
init	     -	   -	    组件初始化。
getWxPage	-	   Page	     返回小程序Page对象。
$getComponent	path(String)	wepy.component	通过组件路径返回组件对象。
$invoke	   com(String/wepy.component), method(String), [args]	-	调用其它组件方法
$broadcast	evtName(String), [args]	-	broadcast事件。
$emit	evtName(String), [args]	-	emit事件。
$apply	fn(Function)	-	准备执行脏数据检查。
$digest	-	-	脏检查。
# QAP授权

[QAP授权](https://mqap.open.taobao.com/doc.htm?spm=a219a.7386653.0.0.237e669avyQepo#?treeId=428&platformId=61&docType=1&docId=107220)



# QAP应用优化

## 优化指南

### 减少 dom 嵌套层数
部分机型设备上，深层次的嵌套可能会导致堆栈溢出。因此要尽可能地减少 dom 嵌套层级，一般页面嵌套层级需控制在 14 层以内。

* 避免不必要的层级嵌套
* 优化组件，使用无状态组件
* 优化布局写法，巧用Flexbox布局

### 拆分你的组件
* 复杂状态与简单状态不要共用一个组件
* 严格区分容器组件与UI组件

一个普通的长列表，其每一项由 ListItem 组件构成，而一个复杂的 ListItem ，可能包含若干个 PureComponent 组成的组件。

```
// 伪代码展示
<ListView>
	<ListItem />
	<ListItem>
		<PureModA />
		<PureModB />
	</ListItem>
</ListView>
```

### 使用 shouldComponentUpdate 提升 diff 性能

为了提高页面运算效率，使用shouldComponentUpdate提升底层diff性能。

PureComponent 涵盖的场景有限，在 PureComponent 不适用的情况下，你需要使用 shouldComponentUpdate ，避免无效 diff。


### 使用 PureComponent
#### PureComponent 原理
当组件更新时，如果组件的 props 和 state 都没发生改变， render 方法就不会触发，省去 Virtual DOM 的生成和比对过程，达到提升性能的目的。PureComponent 实现了 shallowEqual ，自动帮我们做了一层浅比较。

shallowEqual 会比较 Object.keys(state | props) 的长度是否一致，每一个 key 是否两者都有，并且是否是一个引用，也就是只比较了第一层的值，确实很浅，所以深层的嵌套数据是对比不出来的。

使用场景
如果你的 ListItem 组件，内部都是扁平化的 props 或 state，那么可以使用。

```
import { createElement, PureComponent, PropTypes } from 'rax';
import { View, Image, Link, Checkbox } from 'nuke';
class ListItem extends PureComponent {
  constructor(props) {
    super(props);
  }
  render() {
    const { text, pic } = this.props;
    return (
      <View>
          <Text>{text}</Text>
          <Image source={{ uri: pic }} />
      </View>
    );
  }
}
```

## 使用社区类库

如果您对 [npm 社区](https://www.npmjs.com/?spm=a219a.7386653.0.0.3c5e669aVlkzpd) 比较熟悉，也可以使用 npm 社区开源类库，但具有一定的使用前提。

以 moment.js 为例，该类库是一个非常优秀的日期辅助类库，但会修改 String.prototype.toString() 方法，**在千牛 QAP 应用内，这个操作将会污染其他应用的运行时，因此是不允许的，使用时将直接抛错**

常用的社区三方类库资源列表：

属性 | 是否支持 | 	说明
:---: | :---: | :---
animate-value | 	√	
moment | 	x	 | 可以使用 nuke-biz-moment 代替

目前项目里主要是使用nuke-biz-moment，具体使用如下：
### API
[详细介绍](https://mqap.open.taobao.com/doc.htm?spm=a219a.7386653.0.0.237e669avyQepo#?treeId=428&platformId=61&docType=10&docId=265)
#### 格式化类

```
// 按照指定格式格式化
moment('2017/5/22').format('YYYY/MM/DD hh:mm:ss')  // 2017/08/17 13:52:43
```

#### 获取类

```
moment('2017/5/22').year() 
moment('2017/5/22').month() 
moment('2017/5/22').date() 
moment('2017/5/22').second()
moment('2017/5/22').minute()
moment('2017/5/22').format()

```
#### 检验类

```
moment('2017/35/22').isValid() // false
moment('2017/10/20').isAfter('2017/10/19'); // true

```

#### 计算类

```
// 减几天
moment('2017/07/22').addDay(-2).format('YYYY/MM/DD')} // 2017-07-20

// 月份有多少天
moment("2012/02", "YYYY/MM").daysInMonth() //29

```


## 端差异问题

如果你从理解 weex 在客户端渲染原理出发去思考问题，就会得到以下的一些结论：

- native 与 h5 的差异是一定存在的
- iOS 和 Android 的端差异是一定存在的

我们致力于抹平差异，即交互行为和其他端相同或相似，api 保持尽可能一致。如果抹平端差异与 native 交互规范存在冲突，那么我们的原则是：遵循端的交互行为规范，仅 api 一致。

### box-shadow 表现
box-shadow 暂不支持 Android。

### z-index 表现
Weex 目前不支持 z-index 设置元素层级关系，但靠后的元素层级更高，因此，对于层级高的元素，可将其排列在后面。

所以想使用 z-index 实现浮层效果，将会有非常大的局限性， 可以查看 Dialog 组件文档，具有一定的使用条件。

### overflow
如果定位元素超过容器边界，在 Android 下超出部分将不可见。** Android 端元素 overflow 默认值为 hidden，且目前暂不支持设置 overflow: visible**。

### 图片相关组件必须设置宽高才能使用
native 容器必须计算宽高后才能绘制，因此图片需要预先指定宽高。

### h5渐进增强
h5 样式和表现力更佳，如果你想要做渐进增强，可以直接写 native 不支持的样式，native端不会生效。


## 优化 iconfont

Nuke提供了[iconfont API](https://mqap.open.taobao.com/doc.htm?spm=a219a.7386653.0.0.3c5e669aVlkzpd#?_tb_token_=e7ee7543dd7ed&docType=10&docId=246)，方便开发者设置 iconfont，但由于网络加载等原因，首次加载的 Iconfont 可能要在下次进入插件才能生效。

在千牛 QAP 应用内，为了避免这个问题，你可以把用到的 Iconfont 做内置。

假设你使用的 Icon 来自 [iconfont.cn](https://iconfont.cn/?spm=a219a.7386653.0.0.3c5e669aVlkzpd) , 其远程地址为：https://at.alicdn.com/t/font_1474198576_7440977.ttf?version=1.0

```
Iconfont({name:"iconfont1",url:"https://at.alicdn.com/t/font_1474198576_7440977.ttf?version=1.0"})
```

则内置方案如下：

添加 iconfont 声明
打开 qap.json 文件，添加 iconfonts 字段

```
 {
	"appKey":"...","version":"1.0",
	"pages":[...
	],
	"iconfonts":[
		{
			"localpath":"font_1474198576_7440977.ttf",
			"url":"http://at.alicdn.com/t/font_1474198576_7440977.ttf"
		}
	]
 }

```
 
 
其中 localpath 为可选属性，推荐该字段的值和url中的名字保持一致，如上面的代码所示。

把 iconfont 文件放到项目工程中，随项目发布
在 qap 工程的目录下新建 iconfont 目录，把文件下载到此目录，并且命名为font_1474198576_7440977.ttf。

![icon](http://img.alicdn.com/tfs/TB1wCrLPXXXXXbaXFXXXXXXXXXX-1298-518.png)

上述步骤后，再执行打包命令，打出的 zip 包就会包含字体文件，在安装到千牛客户端时会放置到缓存目录，加速 iconfont 的显示。

还可以使用nuke的Iconfont组件。

```
import { Iconfont} from 'nuke';

componentDidMount() {
       Iconfont({
            name: "explorerIconlist", // 为 Iconfont 起个名
            url: "https://at.alicdn.com/t/font_376560_c9dtnthv0ybvs4i.ttf"  //复制 url 中 xxx.ttf ，作为 Iconfont 组件的 url
        });

    }
  render(){
  	return(
  			<View>
  				<Text style={{ fontFamily: 'explorerIconlist', fontSize: '46rem', color: '#3089DC', paddingRight: 10 }}>&#xe671;</Text> : <Text style={{ fontFamily: 'explorerIconlist', fontSize: '46rem', color: '#999999', paddingRight: 10 }}>&#xe660;</Text>
          </View>
  		)
  }

```


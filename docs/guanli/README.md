# 页面管理

## 页面的生命周期

前面介绍了页面的概念，这里介绍下QAP应用（QAPApp）中 页面的生命周期，以及开发者可以利用这些生命周期事件做些什么。

### 基础生命周期事件
一个页面最基本的生命周期包括以下四种：

事件 | 级别	 | 描述 
:--- | :---: | :--
WillAppear |	页面级别 |		当前页面即将进栈时触发
DidAppear |		页面级别 |		当前页面已经进栈时触发
WillDisappear |		页面级别 |		当前页面即将出栈或者下一个页面即将进栈时触发
DidDisappear |		页面级别 |		当前页面已经出栈或者下一个页面已经进栈时触发

以下是监听上述四个生命周期事件的代码：

```
import {createElement, Component, render} from 'rax';
import {Link, View, Text, TouchableHighlight, Button, Navigator} from 'nuke';
import QAP from 'QAP-SDK';

class PageEventsDemo extends Component {
    constructor(props) {
        super(props);

        QAP.on('Page.WillAppear', ()=> {
            console.log('PageEventsDemo Page.WillAppear.');
        });

        QAP.on('Page.DidAppear', ()=> {
            console.log('PageEventsDemo Page.DidAppear.');
        });

        QAP.on('Page.WillDisappear', ()=> {
            console.log('PageEventsDemo Page.WillDisappear.');
        });

        QAP.on('Page.DidDisappear', ()=> {
            console.log('PageEventsDemo Page.DidDisappear.');
        });
    }
    
    render() {
        return (
            <View>
                <Button onPress={()=> {
                    QAP.navigator.push({url: './image.js'});
                }}>push</Button>

                <Button onPress={()=> {
                    QAP.navigator.pop();
                }}>pop</Button>
            </View>
        );
    }
}

module.exports = PageEventsDemo;

render(<PageEventsDemo/>);
```
当打开该页面时，会先后输出两行日志：

```
PageEventsDemo Page.WillAppear.
PageEventsDemo Page.DidAppear.
```
点击push按钮打开下一个页面会输出：

```
PageEventsDemo Page.WillDisappear.
PageEventsDemo Page.DidDisappear.
```
再由下一个页面pop返回到该页面时，会先后输出两行日志：

```
PageEventsDemo Page.WillAppear.
PageEventsDemo Page.DidAppear.
```
最后由当前页面点击返回、pop出栈，会输出：

```
PageEventsDemo Page.WillDisappear.
PageEventsDemo Page.DidDisappear.
```
以上就是页面的四种基础生命周期事件的触发场景，开发者可以在页面切换过程中做一些局部的刷新动作，以保持信息的一致性。

#### 使用示例
这里利用Page.DidAppear和Page.WillDisappear两个页面生命周期事件实现页面切换时的数据同步和刷新。

首先，我们创建一个空的列表页面，并在Page.DidAppear事件触发时捞取用户列表进行展示：

```
'use strict';

import {createElement, Component, render} from 'rax';
import {View, Text, Modal, Slider, ListView, TouchableHighlight, Image, Button} from 'nuke';
import QN from 'QAP-SDK';

class DemoList extends Component {
    constructor(props) {
        super(props);

        this.state = {
            'list':[],
        };

        this.addNavButton();

        QN.on('Page.DidAppear', ()=> {
            QN.sessionstore.get({
                query: {
                    key: 'userList'
                }
            }).then(result => {
                if (result.data.userList) this.setState({'list':result.data.userList});
            }, error => {
                console.log('QN.sessionstore.get userList : ', error);
            });
        });
    }

    addNavButton = () => {
        QN.navigator.addButton({
            query: {
                iconName: 'add',
                text: '添加',
                tapEvent: 'Page.addUser', // 或 `share`
            }
        }).then(result => {
            console.log(result);
        }, error => {
            console.log(error);
        });

        QN.on('Page.addUser', ()=> {
           QN.navigator.push({url:'./addUser.js', title:'添加用户'});
        });
    }

    render() {
        return (
            <ListView 
                style={styles.listContainer}
                dataSource={this.state.list} 
                renderRow={this.cellForRow}>
            </ListView>
        );
    }

    cellForRow = (item, index) => {
        return <TouchableHighlight 
            style={styles.listCell} 
            onPress={() => {this.didSelectCell(item, index)}}>
            <View style={styles.listCellLeftView}>
                <Image style={styles.listCellImage} source={{uri:item.avatar}} />
                <Text style={styles.listCellText}>{item.name}，{item.gender}，{item.age}岁</Text>
            </View>
            
            <Button style={styles.listCellButton}>编辑</Button>
        </TouchableHighlight>;
    }

    didSelectCell = (item, index) => {
        QN.navigator.push({url:item.url, title:item.name});
    }
}

const styles = {
    listContainer: {
        flex: 1,
        backgroundColor: '#E1EFF8',
    },
    listCell: {
        flex: 1,
        flexDirection: 'row',
        justifyContent: 'space-between',
        'backgroundColor': '#E1EFF8',
        'backgroundColor:active': '#cccccc',
        height: '110rem',
        borderBottomStyle: 'solid',
        borderBottomColor: '#dcdde3',
        borderBottomWidth: '1rem',
    },
    listCellLeftView: {
        flex: 1,
        flexDirection: 'row',
    },
    listCellImage: {
        marginLeft: '20rem',
        marginTop: '20rem',
        width: '80rem',
        height: '80rem',
    },
    listCellText: {
        marginLeft: '40rem',
        marginTop: '40rem',
    },
    listCellButton: {
        marginRight: '20rem',
    },
};

render(<DemoList />);

export default DemoList;
```

在上述代码中，我们在导航栏添加了一个按钮，点击跳转到新增用户页面：

```
'use strict';

import {createElement, Component, render} from 'rax';
import {View, Text, Button, TextInput, ScrollView, Modal, Image, Switch} from 'nuke';

import QN from 'QAP-SDK';

class AddUserDemo extends Component {
    constructor(props) {
        super(props);

        this.state = {
            'name': '',
            'age': '',
            'gender':'男',
            'avatar': 'http://is1.mzstatic.com/image/thumb/Purple128/v4/33/5a/d1/335ad132-aba9-1c34-ca95-83d3f7c42d04/source/175x175bb.jpg',
        };

        QN.on('Page.WillDisappear', ()=> {
            this.onPageWillDisappear();
        });
    }

    render() {
        return (
            <ScrollView style={styles.container}>
                <View style={styles.hContainer}>
                    <Text>姓名：</Text>
                    <TextInput style={styles.nameInput} onInput={this.onNameInput}></TextInput>
                </View>
                <View style={styles.hContainer}>
                    <Text>性别：    </Text>
                    <Switch defaultChecked={false} onValueChange={(value) => this.onSwitchChange(value)}/>
                    <Text>    {this.state.gender}</Text>
                </View>
                <View style={styles.hContainer}>
                    <Text>年龄：</Text>
                    <TextInput style={styles.ageInput} maxLength="3" value="20" onInput={this.onAgeInput}></TextInput>
                </View>
            </ScrollView>
        );
    }

    onSwitchChange(value) {
        if (value) {
            this.setState({'gender':'女'});
            this.setState({'avatar':'https://lh3.ggpht.com/ZnMt-3LCs0BgjF27il9SZn8eZFJshGi6ktCcEukUkwXsmq-COWg6chUBd3YG7o10MZUh=w300'});
        } else {
            this.setState({'gender':'男'});
            this.setState({'avatar':'http://is1.mzstatic.com/image/thumb/Purple128/v4/33/5a/d1/335ad132-aba9-1c34-ca95-83d3f7c42d04/source/175x175bb.jpg'});
        }
    }

    onNameInput = (t) => {
        console.log('onNameInput : ', t.value);
        this.setState({'name':t.value});
    }

    onAgeInput = (t) => {
        console.log('onAgeInput : ', t.value);
        this.setState({'age':t.value});
    }

    onPageWillDisappear = () => {
        QN.sessionstore.get({
            query: {
                key: 'userList'
            }
        }).then(result => {
            var userInfo = {
                'name':this.state.name,
                'age':this.state.age,
                'gender':this.state.gender,
                'avatar':this.state.avatar,
            };

            var userList;
            if (result.data.userList) {
                userList = result.data.userList;
            } else {
                userList = [];
            }

            userList.push(userInfo);

            QN.sessionstore.set({
                query: {
                    userList: userList
                }
            }).then(result => {
                console.log('QN.sessionstore.set userList : ', result);
            }, error => {
                console.log('QN.sessionstore.set userList : ', error);
            });
        }, error => {
            console.log('QN.sessionstore.get userList : ', error);
        });
    }
}

const styles = {
    container: {
        flex: 1,
        backgroundColor: '#F5FCFF',
        paddingTop: '20rem',
    },
    hContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingLeft: '20rem',
    },
    nameInput: {
        borderStyle: 'solid',
        borderColor: '#dcdde3',
        borderWidth: '1rem',
        width: '300rem',
        height: '80rem',
        'margin':'40rem'
    },
    ageInput: {
        borderStyle: 'solid',
        borderColor: '#dcdde3',
        borderWidth: '1rem',
        width: '100rem',
        height: '80rem',
        'margin':'40rem'
    },
};

render(<AddUserDemo />);

export default AddUserDemo;

```

当从新增用户页面返回到上一个页面时，在Page.WillDisappear触发时会保存新增的用户，供上一个列表页面实时刷新，效果如下图所示：

![演示](https://img.alicdn.com/tfs/TB1lrpiX.6FK1Jjy1XdXXblkXXa-308-531.gif)

可以通过如上代码来监听前后台切换事件，开发者可以利用这些事件做一些数据存储动作，或者取消定时器等操作。
需要注意的是，当进入后台的事件触发后，只有一小段时间可以执行代码，所以不要在这里做过重的操作。

### 生命周期事件代理

以下是QAP提供给开发者对相关生命周期事件的感知机制，如果开发者想要代理这些事件，可以利用QAP提供的事件代理机制，包括页面的返回、关闭和刷新。

```
QAP.on('Page.back', ()=> {
    console.log('on PageEventsDemo Page.back.');
});

QAP.on('Page.close', ()=> {
    console.log('on PageEventsDemo Page.close.');
});

QAP.on('Page.reload', ()=> {
    console.log('on PageEventsDemo Page.reload.');
});
```
开发者可以通过以上代码监听代理返回、关闭和刷新事件，当用户触发相应操作时会输出如下日志：

```
on PageEventsDemo Page.back.
on PageEventsDemo Page.close.
on PageEventsDemo Page.reload.
```

这些事件代理功能可以让开发者做一些体验上的优化，比如在重要操作界面避免用户误操作返回、导致数据丢失。
需要注意的是，当开发者代理了以上事件后，也需要给出合理的后续交互流程，否则开发者的应用在交互评审时很可能会无法通过，导致无法正式发布该应用。
这种场景下，开发者可以利用QAP提供的页面栈管理功能来实现后续的交互。

[协议api入口](https://mqap.open.taobao.com/doc.htm?spm=a219a.7386653.0.0.237e669avyQepo#?treeId=428&platformId=61&docType=1&docId=107263)



## 页面栈管理

一个QAP应用包含一个或者多个页面，这些页面会被放到该应用的页面栈中进行管理。
在页面栈的基础上，QAP提供了丰富的API来支持开发者灵活地管理页面之间的跳转。

### push与pop

这是基础的2个页面栈管理的API，分别是打开一个新页面、关闭当前页面



```
<!--打开一个新页面-->
QN.navigator.push({url:url});


<!--关闭当前页面-->
QN.navigator.pop();
```

效果如下图所示：

![push](http://img01.taobaocdn.com/top/i1/LB1WferXxTxLuJjy1XcXXb.gXXa)

这里的url可以是如下几种格式，对应qap.json配置文件的pages字段中的每个元素的url字段：

页面地址 |	本地/远程 |	 页面类型
:--- | :---: | :---
qap:///index.js | 	本地	 | Native
https://m.taobao.com/ | 	远程 | 	H5
https://host/index.js | 	远程 | 	Native，后缀也可以是wx
https://host/?_wx_tpl=http://host/index.js | 远程 | Native
https://host/?wh_weex=true | 	远程 | 	由服务端返回的Content-Type决定

需要注意的是在第一个页面地址示例中，qap协议后面跟着三个斜杠、无需填写host字段信息，因为QAP会根据路径到对应的应用目录下寻找文件。
同时，这也是一个保留特性，请开发者务必遵循。

### push进阶

#### 携带标题和页面参数

除了基本的url参数外，push这个API还提供了[其它扩展参数](https://g.alicdn.com/x-bridge/qap-sdk/2.1.3/docs/api/api-navigator.html?spm=a219a.7386653.0.0.3c5e669aVlkzpd#api-%E8%B0%83%E7%94%A8%E5%85%A5%E5%8F%82)，比如支持在push时直接设置下一个页面的标题，并且传递一些参数：

```
<Button style={styles.testButton} onPress={() => {this.pushWithTitleAndQuery('./navigator.js', 'pushwithTitle', {a:'1'})}}>pushWithTitle</Button>
    ...
    pushWithTitleAndQuery = (url, title, query) => {
        QN.navigator.push({
            url:url,
            title:title,
            query:query
        });
    }
 
 
 <!--项目预览，直接跳转到TradeDetailNewPage.js页面-->
 
  QN.navigator.push({
    url: 'qap:///TradeDetailNewPage.js',
        query: { tid: tid },
        settings: {
            animate: true,
            request: true,
        },
    success(result) {
    },
    error(error) {
        }
    }).then(result => {
    }, error => {});
 
```

#### clearTop

除此之外，QAP的页面栈还支持在push一个新页面进栈时，先清空现有页面栈，将新页面作为应用的第一个页面（即栈底）展示：

```
QN.navigator.push({
    url:url,
    title:title,
    query:query,
    settings:{animated:true, clearTop:true}
});
```

通过在settings参数中设置clearTop的值可以实现该效果。

### pop进阶

####  pop(n)

通过制定要出栈的页面数目来返回：

```
<!--关闭当前页面到上一个页面-->

QN.navigator.pop()

<!--传递参数，可以实现关闭n个页面-->

QN.navigator.pop({query:{pages:n}});

```

#### popTo

通过在query中设置index或者url参数可以返回到指定页面，其中index从0开始，并且优先级高于url。

```
<!--可以是页面数量-->
QN.navigator.popTo({query:{index:n}});

<!--可以是URL链接-->
QN.navigator.popTo({query:{url:url}});
    
```

### 关闭和刷新

页面可以监听页面事件进行自定义操作，back、close和reload等事件来实现自定义的交互。

监听的事件 | 相应的API
:--- | :---
back | 	QN.navigator.pop();
close | 	QN.navigator.close();
reload | 	QN.navigator.reload();

### 打开其它应用

使用`QN.navigator.push()`功能还可以拨打电话、发短信、打开手机淘宝等其它应用：
 
```

QN.navigator.push({url:'tel:13888888888'});
QN.navigator.push({url:'sms:13888888888'});
QN.navigator.push({url:'taobao://'});

```

更加详细说明[地址链接](https://mqap.open.taobao.com/doc.htm?spm=a219a.7386653.0.0.237e669avyQepo#?treeId=428&platformId=61&docType=1&docId=107279)


## 页面能力路由

用户在不同的业务场景下打开同一个应用时，开发者可能期望展示不同的页面给用户浏览和操作。页面能力路由的就是负责打开插件是把不同的业务场景与插件配置的对应页面关联起来。
比如点击工作台数字面板的“待付款”会根据能力配置打开交易管理插件中的“待付款”列表页。

![路由](http://img01.taobaocdn.com/top/i1/LB1CCxRXoUIL1JjSZFrXXb3xFXa)


开发者可以在qap.json文件中的pages字段中的每个页面节点信息中设置对应的capability值，示例如下：

```
{
    "appKey":"0000000",
    "version":"1.0",
    "pages":[
       {
	      "default": true,
	      "url": "qap:///index.js"
	    },
	    {
	      "capability": "tradeDetail",
	      "url": "qap:///TradeDetailNewPage.js"
	    },
	    {
	      "capability": "event_tradeDetail",
	      "url": "qap:///TradeDetailNewPage.js"
	    },
	    {
	      "capability": "tradeList",
	      "url": "qap:///index.js"
	    },
	    {
	      "capability": "event_tradeList",
	      "url": "qap:///index.js"
	    },
	    {
	      "capability": "refundDetail",
	      "url": "qap:///index.js"
	    },
	    {
	      "capability": "preIndex",
	      "url": "qap:///PreIndex.js"
	    }
    ]
}
```

capability的取值范围和场景如下：

capability | 场景 | 	中文名称 | 说明
:--- | :---: | :---: | :---
itemList | 	出售中（数字） | 	商品列表	 | 附带参数“itemStatus”:“onsale”
 | 橱窗中 （数字） | 	商品列表 | 	附带参数“itemStatus”:“hasshowcase”
 | 出售中宝贝（数字）	 | 商品列表 | 	附带参数“itemStatus”:“onsale”
 | 已橱窗推荐 （数字）	 | 商品列表	 | 附带参数“itemStatus”:“hasshowcase”
 | 昨日成交宝贝（数字） | 	商品列表 | 	附带参数“itemStatus”:“onsale”
tradeList | 	待付款（数字） | 	交易列表 | 	附带参数“tradeStatus”:“WAIT_BUYER_PAY”
 | 待发货（数字） | 	交易列表	 | 附带参数“tradeStatus”:“WAIT_SELLER_SEND_GOODS”
 | 退款中（数字） | 	交易列表 | 	附带参数“tradeStatus”: “TRADE_REFUND”
 | 今日子订单数（数字） | 	交易列表	
 | 昨日订单数（数字）	 | 交易列表	
 | 待评价（数字） | 	交易列表 | 	附带参数“tradeStatus”: “WAIT_BUYER_APPRAISE”
 | 今PC子订单（数字）	 | 交易列表	  
itemDetail | 	商品消息（消息） | 	商品详情	
tradeDetail | 	交易消息（消息） | 	交易详情	
refundDetail | 	退款消息（消息） | 	退款详情	

如果QAP根据capability寻找不到匹配的页面，则会尝试去寻找带有"default":true的默认页面，如果还搜索不到就会寻找pages字段中的第一个页面信息来展示。
 

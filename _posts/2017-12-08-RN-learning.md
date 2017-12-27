---
layout:     post
title:      "RN-learning"
subtitle:   "RN基础学习(长期更新)"
date:       2017-12-27 22:30:00
author:     "Mickey"
header-img: "img/post-bg-e2e-ux.jpg"
tags:
    - 前端开发
---

很早之前就想学习RN了，最近终于有时间，陆陆续续的开始学习，这里是一些知识点，用以温故知新～

* react-native中存在一个forceUpdate(function callback)方法，如果开发者由于某种原因，使得UI中可变数据的来源必须从state和prop外获取，那么可以手动调用forceUpdate()方法，需要注意这个方法不会调用shouldComponentUpdate()方法来检查是否更新

* 从0.44版本开始，Navigator被从react native的核心组件库中剥离到了一个名为react-native-deprecated-custom-components的单独模块中，现在社区主推的导航包是`React Navigation`，另外，如果仅仅是在ios上开发，可以选用`NavigatorIOS`这个包

* RN中textAlign不能用在View上，justifyContent: flex-start/flex-end/center/space-around/space-between，alignItems: flex-start/flex-end/center/stretch

* RN里组件可以定义多个组件样式，后面的样式会覆盖前面的，如果样式需要动态改变的话，可以和state挂钩

	```js
	style={{ styles.style1, styles.style2 }}
	style={{ styles.style1, {color: this.state.color}}}
	```

* 我们知道RN中的数值都是逻辑像素，那么如何让一个组件的边框只有一个实际像素的宽度呢？

	```js
	let dpr = require('PixelRatio').get();
	borderWidth: 1/dpr
	```

* View组件实现transform不好，但是很多继承View的组件可以使用，比如Text

	```js
	transform: [ {rotate: '45deg'}, {scale: 2} ]
	```
	
* View组件有一个属性removeClippedSubviews，这是一个用于性能优化的属性，通常用于ScrollView或者ListView中，当组件有很多子组件的时候，将这个属性设为true让RN释放不在视口的子组件，类似于懒加载，要让此属性生效，需要将组件和子组件的overflow都设为hidden

* View组件有一个pointerEvents属性，用于决定由组件的什么部位响应触摸事件，某些继承于View的组件支持性可能没有那么好，pointerEvents为null的时候为默认行为，none的时候本组件和子组件触摸事件由父组件处理，box-none的时候本组件由父亲组件响应，子组件由本组件响应，box-none的时候本组件和子组件的事件由本组件响应

* Dimensions只能获取应用启动时的长度和宽度，onLayout可以在组件加载和旋转的时候动态更新

	```js
	const dimensions = require('dimensions');
	{
		let { width, height } = dimensions.get(window); 
	}
	...
	_onLayout(event) {
		let { x, y, width, height } = event.nativeEvent.layout;
	}
	```

* Image不是继承于View的组件，但是存在onLayout属性，可以用于获得图片在不同的显示style下的width和height，另外，Image还有onLoad，onLoadStart，onLoadEnd三个属性，onLoad只有成功加载才会调用，而onLoadEnd不管是否成功加载都会执行，但是这三个onLoad函数都只有一个event.timestamp有用，指出了事件发生的时间
	
	```
	let {x, y, width, height} = event.nativeEvent.layout;
	```

* 可触摸类型组件大致分为TouchableOpacity和TouchableHighlight两种，需要注意的是，可触摸类型组件只能有一个子组件，如果需要有多个子组件，可以用一个view包装，然后让view成为可触摸组件的子组件。
	* <TouchableOpacity activeOpacity={}>，默认的activeOpacity为0.2，也可以手动设置为0 - 1之间的小数
	* TouchableOpacity和TouchableHighlight的一些其他的属性，onPress，onLongPress，onPressIn，onPressOut，delayLongPress（按多少ms触发onLongPress，默认是500ms），delayPressIn（手指接触按钮多少ms出发onPressIn，默认为0），delayPressOut（手指接触按钮多少ms出发onPressOut，默认为0），另外pressRetentionOffset是对象类型的属性，{ top: number, bottom: number, left: number, right: number }，用于节约内存，在手指离开touchable控件多少距离外，可触摸控件变为不可触摸，`需要注意的是这个对象属性只有在父组件为不可滚动的时候才有效`

* Text组件嵌套的时候，子组件不能覆盖父组件的样式，只能在此基础上增加，如果在Text中需要换行显示，则需要使用{'\r\n'}

* Text上下左右居中显示，可以用textAlign: 'center'和lineHeight等于height来实现，也可以将根组件设为flex: 1，然后在Text组件外包一层View组件，然后View组件justifyContent和alignItems均为center来实现

* Text在fontSize和height不同的情况下在android和ios上的表现有区别，然后border的实现在两种平台上也有区别，android端需要在Text组件外包裹一个View组件用以展示边框，如果为了统一，可以都采用这种写法

* TextInput组件的属性(TextInput在两个平台上也有不同的属性，需要的时候可以查询文档)
	* autoCapitalize，字符串类型，none/sentences/words/characters
	* autoFocus，boolean类型，用于决定是否自动focus焦点
	* defaultValue，组件内的默认值，这个比用value实现效果好，value可能会闪屏一下
	* editable，boolean，组件是否可以输入
	* keyboardType，字符串类型，android和ios都支持的default, numeric, email-address
	* maxLength，数字类型，用于限制input最长输入数
	* multiline，布尔类型，为true时，可以多行输入as
	* secureTextEntry，布尔类型，用于决定输入框是否用于输入密码，password好像没用

* RN和react绑定DOM元素的方法一样，RN的组件包含一个setNativeProps函数，可以用于直接修改组件的属性，一般不建议这么用，流程是先用ref绑定好组件，再this.component.setNativeProps({ property: value })来使用
* RN提供ClipBoard组件来写入剪切板和从剪切板中读出

	```js
	import { ClipBoard } from 'react-native';
	
	...
	
	ClipBoard.setString('mickey');
	
	ClipBoard.getString().then(pasteText => { ... })
	```

* 我们都知道在H5页面上，有sessionStorage和localStorage两个浏览器端的API用于存储，RN也有类似的API--AsyncStorage

	```js
	import { AsyncStorage } from 'react-native';
	
	...
	
	AsyncStorage.setItem(key, value); // 这里需要注意key和value都要为string，不然会报错
	
	AsyncStorage.getItem(key)
	.then(value => {
		...
	})
	.catch(err => {
		...
	})
	```

* ScrollView和FlatList、SectionList（ListView是低版本用的）是RN中几个滚动组件，组件的属性都比较多，包含滑动到边界， 滑动过程中的事件监听，是否允许放大，事件监听时延等，具体可以使用的时候再进行查看

* SrollView的组件成员函数，scrollTo({x, y, animated})，animated代表过去的时候是否需要动画

* RN0.43之后，退出了高性能滚动列表FaltList，详细见官方文档，这里给出一个demo，[参考资料](http://blog.csdn.net/u011272795/article/details/74359305)

	```js
    <FlatList
  refreshing={this.state.isRefreshing}
  onRefresh={() => this.updateNum()}
  ItemSeparatorComponent={ItemSeparator}
  getItemLayout={(data, index) => ({ length: 40, offset: 41 * index, index})}
  initialNumToRender={ Math.ceil(totalHeight / 40) }
  ListFooterComponent={<View style={styles.header}/> }
  ListHeaderComponent={<View style={styles.header} />}          
  data={this.flatListTest}
  renderItem={({ item }) => <Text style={styles.flatListItem}>{item.key}</Text> } />
	```
	
	refreshing可以用于在向服务端请求数据的时候在顶部展示一个默认的加载菊花图，onRefresh的函数在滚动到顶部下拉的时候会执行，可以在这里去请求服务端数据，ItemSeparatorComponent描绘的是list之间的分割组件，ListHeaderComponent，ListFooterComponent是顶部和底部的组件（这三个组件没必要新写一个class，直接在当前class里写一个函数返回一个组件就行），initialNumToRender指代第一次渲染多少item，getItemLayout对组件滚动优化很有作用，指代的是每一个item的list中的位置，data是列表的数据源，renderItem是渲染的Item组件

* SectionList和FlatList一样，是RN0.43之后发布的高性能分组组件，如果不用分组的话，FlatList无疑是更好的选择，SectionList大多数属性和FlatList相同，下面来讲一下不同的地方，[参考资料](http://www.jianshu.com/p/6302c4d48b97)

	```
	<SectionList 
	SectionSeparatorComponent={() => this.genSectionHeaderSeparator()}
	initialNumToRender={20}
	refreshing={this.state.isRefreshing}
	onRefresh={() => this.updateNum()}
	ItemSeparatorComponent={ItemSeparator}
	ListFooterComponent={() => <View style={styles.header} />}
	ListHeaderComponent={<View style={styles.header} />}
	sections={this.sectionTest}
	renderSectionHeader={({ section }) => <Text style={styles.sectionHeader}>{section.key}</Text> }
	renderItem={({ item }) => <Text style={styles.flatListItem}>{item.key}</Text> }
	/>
	```
	
	FlatList的data是一个由对象组成的数组，SectionList的sections是由分组对象组成的数组，每一个分组对象由分组名称和分组内容组成，renderSectionHeader代表渲染一个分组标题，renderItem用于渲染分组中的item，如果不同的分组需要不同的渲染方式，应该在分组对象中用renderItem表示出来，SectionSeparatorComponent在分组头部和尾部渲染一个分割组件

* ProgressViewIOS组件在父亲组件为flex布局的情况下，不会正常显示，需要给组件设置如下的样式

	```js
	let totalWidth = require('Dimensions').get(window).width;
	
	{
		width: totalWidth,
		position: 'absolute',
		top: 0,
	}
	```
	
* PanResponder，RN中用于提供手势监控的控件，首先需要使用PanResponder.create()创造一个watcher，create方法的参数是各种事件的回调，常用的事件回调为onPanResponderGrant和onPanResponderMove，分别用于监听单点和移动事件，回调函数有两个参数，一般用gestureState来获取x0，y0，moveX，moveY即可，需要将watcher.panHandlers透传到手势检测空间上，检测控件可以为一个界面上显示的控件，也可以是一个背景为transparent的控件，看需求而定

	```js
	componentWillMount() {
	  this.watcher = PanResponder.create({
	    onStartShouldSetPanResponder: () => true,
	    onPanResponderGrant: this.onPanResponderGrant,
	    onPanResponderMove: this.onPanResponderMove,
	  });    
	}
	
	...
	
	<View 
    	style={styles.touchViewStyle}
        {...this.watcher.panHandlers} />
	```

* NetInfo用于获取当前手机的网络信息
	* NetInfo.getConnectionInfo: 用于获取当前手机的网络情况，返回一个promise
	* NetInfo.addEventListener('change', handler, false): 用于监听手机网络状态的变化情况，需要在 cwum 中销毁监听器
	* NetInfo.isConnected.fetch().done(): 用于判断当前手机是否有网络连接
	* android端存在 NetInfo.isConnectionExpensive来获取当前网络是否计费

* Webview组件可以用于打开一个已经存在的网页，也可以用于打开RN项目中的静态文件

* RN中强制app竖屏显示的方法
    * IOS把info.plist的UISupportedInterfaceOrientations中UIInterfaceOrientationLandscapeLeft、UIInterfaceOrientationLandscapeRight注释掉
    * Android在AndroidManifest中的MainActivity后面增加android:screenOrientation="portrait"

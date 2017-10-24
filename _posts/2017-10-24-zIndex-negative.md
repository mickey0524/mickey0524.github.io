---
layout:     post
title:      "zIndex-negative"
subtitle:   "z-index为负值的元素无法被选中"
date:       2017-10-24 23:30:00
author:     "Mickey"
header-img: "img/post-bg-rwd.jpg"
tags:
    - 前端开发
    - CSS
---

今天撒欢写代码的时候，突然看到运营反馈群里，有同学说，表格中的元素无法选中，PM让我解决一下，emmmmm....

我思考了一下，最近除了修复了一个滚动bug，也没做啥修改啊，然后去看了一下代码，发现👇的代码

```
#wrap-comp {
	position: relative;
	z-index: -10;
}

<div id="header"></div>
<div id="wrap-comp">
	<table border="1">
		<thead>
			<tr>
				<th>标签</th>
			</tr>
		</thead>
		<tbody>
			<tr>
				<td>内容</td>
			</tr>
		</tbody>
	</table>
</div>
```

这样设置可以让id为wrap-comp的位于文档流的底部，滚动的时候，内容不会覆盖在header之上，但是事实证明这样设置后, 元素无法被点击到，hover时也不会显示手形，因为z-index为负值的元素将被放置在body层之下，所以点击和hover事件都被body层覆盖了。

### 解决方法

将wrap-comp的z-index设为0，将header的style设为position:relative; z-index: 1，这样能保证header和wrap-comp两个区域均位于body层之上，于是都能被选中，同时也不会有滚动显示异常的bug

```js
#header {
	position: relative;
	z-index: 1;
}
#wrap-comp {
	position: relative;
	z-index: 0;
}
```

### 总结

这个问题其实非常基础，但是写代码的时候仅仅考虑了实现，于是看到显示没有异常之后，就没有继续思考，导致出现了较为低级的bug，以后写代码的时候，无论需求多小，还是要好好对待～



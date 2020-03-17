> 选择器

- 元素选择器:标签名{}

- id选择器：\#id属性值{}
- 类选择器：.class属性值{}
- 通配选择器：*{}
- 复合选择器(交)：选择器1选择器2选择器N{}
- 分组选择器(并)：选择器1，选择器2，选择器N{}
- 后代元素选择器：祖先元素 后代元素{}
- 子元素选择器：父元素 > 子元素{}
- 伪类选择器：

```css
a:link{
  color: yellow //未访问过的
}
a:visited{
  color: red //访问过的,只能设置字体颜色
}
a:hover{
  color: green //鼠标移入
}
a:active{
  color: black //正在被点击
}
//hover与active可以给其他元素设置
input:focus{
  background-color: orange
}
p::selection{
  background-color: blue
}
```

- 伪元素选择器：

```css
p:first-letter{
  font-size: 20px;
}
p:first-line{
}
p:before{ //结合content样式使用
	content: "before";
  color: purple;
}
p:after{ 
}
```

- 属性选择器：[属性值]{}

```css
p[title]{  
}
p[title="hello"]{ 
}
p[title^="hello"]{ //以hello开头的属性 
}
p[title$="hello"]{ //以hello结尾的属性 
}
p[title*="hello"]{ //属性值包含hello
}
```

- 子元素伪类

```css
p:first-child{
}
p.nth-child(3){  //设置为even,表示偶数位置子元素；设置为odd，表示奇数位置。
}
p.first-of-type{ //
}

```

- 兄弟元素选择器

前一个 + 后一个

前一个 ~ 后面所有

- 否定伪类：not(选择器)

> 优先级

- 内联样式 1000
- id选择器 100
- 类和伪类 10
- 元素选择器 1
- 通配选择器 0
- 继承 没有优先级
- 并集选择器单独计算

- ！important 有最高优先级

> 布局

- border

```css
.box1{
	width: 100px; //内容区宽度
	height: 100px;
	background-color: blue;
	border-width: 10px; //边框宽度,以下三个样式缺一不可。可指定四个方向 4个值：上右下左 3个值： 上 左右 下 2个值：上下 左右
	border-color: yellow;
	border-style: solid; //只指定style，width和color有默认值
	border: 10px red solid; //简写，不能分别指定,border-xxx，不需要的border-left: none		
}
```

- padding

内边距会影响盒子可见框大小，元素背景会延伸到内边距

```html
<style type="text/css">
        .box1{
            width: 100px;
            height: 200px;
            background-color: blue;
            border: red 10px double;
            padding-top: 100px;
        }
        .box2{
            width: 100%;
            height: 100%;
            background-color: yellow;
        }
</style>
<body>
    <div class="box1">
        <div class="box2"></div>
    </div>
</body>
```

盒子可见框宽度=boarder+width+padding

- margin

1. 外边距，盒子与其他盒子的距离，不影响盒子可见框的大小，影响盒子的位置

2. 设置左上外边距，影响盒子的位置

设置右下外边距，影响其他盒子的位置

3. 可以设置为负值

4. 如果只指定左边距或右边距的margin为auto，会将外边距设置为最大值

垂直方向外边距设置为auto，默认为0

左右边距均为auto，两侧外边距设置为相同的值，则会居中

5. 相邻、垂直方向外边距重叠问题

兄弟元素之间相邻外边距会取最大值

父子元素垂直外边距相邻，子元素外边距会设置给父元素，解决方式：添加padding或border，使得垂直外边距不相邻

6. 浏览器有默认样式，去除方法

```css
*{
	margin: 0;
  padding: 0;
}
```

- 内联元素

1. width和height无效
2. 可以设置水平内边距，会影响布局；垂直方向内边距不会影响布局
3. 可以设置边框，但垂直边框不会影响布局
4. 不支持垂直外边距，支持水平外边距，但不会重叠

- display

```css
display:inline //内联
display:block //设置为块元素
display:incline-block //行内块：既可以设置宽高，又不独占一行
display:none //不显示，不占位
```

- visibility

```css
visibility:visible //默认
visibility:hidden //不显示，会占位
```

- overflow

```css
overflow:visible //默认，子元素大小超过父元素内容区，超过的大小会在以外的位置显示
overflow:hidden //溢出的内容不会显示
overflow:scroll //无论溢出，滚动条显示
overflow:auto //如果溢出，滚动条显示
```

- 浮动

块元素在文档流中默认垂直排列，三个div只能自上而下排开。使用float让其脱离文档流。

```css
float: none //默认值，文档流中排列
float: left //脱离文档流，向左侧浮动。下边元素向上移动
```

- BFC

设置元素浮动

设置元素绝对定位

设置为inline-block

设置overflow一个非visible值（推荐overflow:auto)

> 定位

- absolute：

元素会脱离文档流

相对于离他最近的开启了定位的祖先元素进行定位。如果不存在这样的祖先元素，则相对于浏览器窗口进行定位

绝对定位会使元素提升一个层级

绝对定位会修改元素性质，内联元素变成块元素，块元素高度宽度默认被内容撑开

- fixed：

固定定位也是一种绝对定位，但不同的是其永远相对于浏览器窗口定位，并且固定定位不会随滚动条而滚动

- relative：

设置left、right、top、bottom，相对其定位位置进行移动，不会影响其他元素

相对定位会使元素提升一个层级，如果重叠会覆盖

相对定位不会修改元素性质（块、内联）

- static：默认值，未开启定位

> 层级

自定义z-index，数字越大，层级越高，元素必须开启定位

父元素层级再高，也不会盖住子元素

> opacity

0完全透明，1完全不透明

> 背景

- backgroud-image：url(相对路径)

如果背景图片大于元素，默认显示左上角

背景图片小于元素，会平铺

同时指定背景颜色和图片，颜色会作为图片底色

- backgroud-repeat

默认repeat，可设置为no-repeat，repeat-x(水平方向重复)

- background-position

调整背景图片在元素中的位置，可以使用top、right、left、buttom、center中的两个值

也可以指定两个偏移量，如background-position : 100px, 100px（第一个值为水平偏移量）

- background-attachment

默认scroll，背景图片会随着窗口滚动

可以设定为fixed，背景图片定位永远相对于浏览器窗口。
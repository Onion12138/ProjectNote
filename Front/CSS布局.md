## CSS布局

### 水平居中

*方案一*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 100%;
            height: 200px;
            background: red;
            text-align: center
        }
        #child {
            width: 200px;
            height: 200px;
            background: blue;
            display: inline-block;
        }
    </style>
</head>
<body>
<div id="parent">
    <div id="child"></div>
</div>
</body>
</html>
```

注意：如果设置为inline，则width和height属性无效

优点：浏览器兼容性好

缺点：子级元素文本也会居中。解决方案，在子级元素设置text-align

*方案二*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 100%;
            height: 200px;
            background: red;
            text-align: center
        }
        #child {
            width: 200px;
            height: 200px;
            background: blue;
            display: table;
            margin: 0 auto;
        }
    </style>
</head>
<body>
<div id="parent">
    <div id="child">hello</div>
</div>
</body>
</html>
```

margin

一个值：上右下左

二个值：上下 左右

三个值：上 左右 下

display 使用table和block均可

优点：只需要设置子级元素

缺点：若子级元素脱离文档流，则margin属性无效，如将position设置为absolute

*方案三*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 100%;
            height: 200px;
            background: red;
            position: relative;
        }
        #child {
            width: 200px;
            height: 200px;
            background: blue;
            left: 50%;
            transform: translateX(-50%);
            position: absolute;
        }
    </style>
</head>
<body>
<div id="parent">
    <div id="child">hello</div>
</div>
</body>
</html>
```

当前元素设置为绝对定位之后，如果父级元素未开启定位，当前元素相对于页面；父级元素开启定位，当前元素相对于父级元素

优点：父级元素是否脱离文档流，不影响子级元素居中效果

缺点：transform为css3属性，兼容性不好

### 垂直居中

*方案一*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 200px;
            height: 600px;
            background: red;
            display: table-cell;
            vertical-align: middle;
        }
        #child {
            width: 200px;
            height: 200px;
            background: white;
        }
    </style>
</head>
<body>
<div id="parent">
    <div id="child">Onion</div>
</div>
</body>
</html>
```

vertical-align用于设置文本对齐方式，有top，middle和bottom

display属性：table设置当前元素为<table>，tabel-cell设置当前元素为<tr>

优点：兼容性好

*方案二*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 200px;
            height: 600px;
            background: red;
            position: relative;
        }
        #child {
            width: 200px;
            height: 200px;
            background: white;
            position: absolute;
            top: 50%;
            transform: translateY(-50%);
        }
    </style>
</head>
<body>
<div id="parent">
    <div id="child">Onion</div>
</div>
</body>
</html>
```

### 居中布局

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 1000px;
            height: 600px;
            background: red;
            display: table-cell;
            vertical-align: middle;
        }
        #child {
            width: 200px;
            height: 200px;
            background: gray;
            display: block;
            margin: 0 auto;
        }
    </style>
</head>
<body>
<div id="parent">
    <div id="child">Onion</div>
</div>
</body>
</html>
```



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 1000px;
            height: 600px;
            background: red;
            position: relative;
        }
        #child {
            width: 200px;
            height: 200px;
            background: gray;
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%);

        }
    </style>
</head>
<body>
<div id="parent">
    <div id="child">Onion</div>
</div>
</body>
</html>
```

### 两列布局

效果：定宽与自适应

*方案一*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #left, #right {
            height: 300px;
        }
        #left {
            width: 300px;
            background: yellow;
            float: left;
        }
        #right {
            background: green;
            margin-left: 300px;
        }
    </style>
</head>
<body>
    <div id="left">定宽</div>
    <div id="right">自适应</div>
</body>
</html>
```

缺点：自适应的margin-left必须与定宽的宽度相同

以下情景上述方法失效

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #left, #right {
            height: 300px;
        }
        #left {
            width: 300px;
            background: yellow;
            float: left;
        }
        #right {
            background: green;
            margin-left: 300px;
        }
        #inner {
            height: 300px;
            background-color: blue;
            clear: both;
        }
    </style>
</head>
<body>
    <div id="left">定宽</div>
    <div id="right">自适应
        <div id=inner></div>
    </div>
</body>
</html>
```

*方案二*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #left, #right {
            height: 300px;
        }
        #left {
            width: 300px;
            background-color: yellow;
            float: left;
        }
        #right {
            background-color: green;
            overflow: hidden;
        }
    </style>
</head>
<body>
    <div id="left">定宽</div>
    <div id="right">自适应
    </div>
</body>
</html>
```

*方案三*

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            display: table;
            table-layout: fixed;
            width: 100%;
        }

        #left,
        #right {
            height: 300px;
            display: table-cell;
        }

        #left {
            width: 300px;
            background-color: yellow;
        }

        #right {
            background-color: green;
        }
    </style>
</head>

<body>
    <div id="parent">
        <div id="left">定宽</div>
        <div id="right">自适应</div>
    </div>
</body>

</html>
```

### 三列布局

定宽+定宽+自适应

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #left,#center,#right {
            height: 300px;
        }
        #left {
            width: 300px;
            background-color: yellow;
            float: left;
        }
        #center {
            width: 300px;
            background-color: blue;
            float: left;
        }
        #right {
            background-color: green;
            margin-left: 600px;
        }
    </style>
</head>

<body>
        <div id="left">定宽</div>
        <div id="center">定宽</div>
        <div id="right">自适应</div>
</body>

</html>
```

### 圣杯布局

核心：定宽+自适应+定宽

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #left,#right {
            width: 300px;
            height: 300px;
        }
        #left {
            background-color: yellow;
            float: left;
        }
        #center {
            height: 300px;
            background-color: blue;
            margin-left: 300px; 
            margin-right: 300px;
        }
        #right {
            background-color: green;
            float: right;
        }
    </style>
</head>

<body>
        <div id="left">定宽</div>
        <div id="right">定宽</div>
        <div id="center">自适应</div>
       
</body>

</html>
```

缺点：只能把center放在最后

### 双飞翼布局

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            height: 300px;
        }
        #left,#center,#right {
            height: 300px;
            float: left;
        }
        #left,#right {
            width: 300px;
        }
        #left {
            background-color: yellow;
            margin-left: -100%;
        }
        #center {
            background-color: blue;
            width: 100%;
        }
        #right {
            background-color: green;
            margin-left: -300px;
        }
        #inner {
            height: 300px;
            background-color: pink;
            margin-left: 300px;
            margin-right: 300px;
        }
    </style>
</head>

<body>
    <div id="parent">
        <div id="center">
            <div id="inner"></div>
        </div>
        <div id="left"></div>
        <div id="right"></div>
    </div>   
</body>

</html>
```

### 等分布局

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .col1,.col2,.col3,.col4 {
            height: 300px;
            width: 25%;
            float: left;
        }
        .col1{
            background-color: pink;
        }
        .col2{
            background-color: blue;
        }
        .col3{
            background-color: green;
        }
        .col4{
            background-color: yellow;
        }
        
    </style>
</head>

<body>
    <div id="parent">
        <div class="col1">1</div>
        <div class="col2">2</div>
        <div class="col3">3</div>
        <div class="col4">4</div>
    </div>   
</body>

</html>
```



```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            width: 100%;
            display: table;
        }
        .col1,.col2,.col3,.col4 {
            height: 300px;
            display: table-cell;
        }
        .col1{
            background-color: pink;
        }
        .col2{
            background-color: blue;
        }
        .col3{
            background-color: green;
        }
        .col4{
            background-color: yellow;
        }
        
    </style>
</head>

<body>
    <div id="parent">
        <div class="col1">1</div>
        <div class="col2">2</div>
        <div class="col3">3</div>
        <div class="col4">4</div>
    </div>   
</body>

</html>
```

存在间距的情况

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            height: 300px;
            margin-left: -10px;
        }
        .col1,
        .col2,
        .col3,
        .col4 {
            height: 300px;
            width: 25%;
            float: left;
            box-sizing: border-box;
            padding-left: 10px;
        }
        .inner {
            height: 300px;
        }
        .col1 .inner {
            background-color: pink;
        }

        .col2 .inner{
            background-color: blue;
        }

        .col3 .inner{
            background-color: green;
        }

        .col4 .inner{
            background-color: yellow;
        }
    </style>
</head>

<body>
    <div class="parent-fix">
        <div id="parent">
            <div class="col1"><div class="inner"></div></div>
            <div class="col2"><div class="inner"></div></div>
            <div class="col3"><div class="inner"></div></div>
            <div class="col4"><div class="inner"></div></div>
        </div>
    </div>
</body>

</html>
```

### 等高布局

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            display: table
        }
        #left, #right {
            width: 300px;
            display: table-cell
        }
        #left{
            background-color: yellow
        }
        #right{
            background-color: green
        }
    </style>
</head>

<body>
        <div id="parent">
            <div id="left">left</div>
            <div id="right">right</div>
        </div>
</body>

</html>
```

### CSS3的多列布局

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #parent {
            column-count: 4;
            column-width: 300px;
        }
        .col1,.col2,.col3,.col4 {
            height: 300px;
        }
        .col1{
            background-color: pink;
        }
        .col2{
            background-color: blue;
        }
        .col3{
            background-color: green;
        }
        .col4{
            background-color: yellow;
        }
        
    </style>
</head>

<body>
    <div id="parent">
        <div class="col1">1</div>
        <div class="col2">2</div>
        <div class="col3">3</div>
        <div class="col4">4</div>
    </div>   
</body>

</html>
```




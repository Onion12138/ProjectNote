## Vue

### 案例驱动入门篇

#### 入门案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">{{content}}</div>
    <div>{{content}}</div>
    <script>
        var app = new Vue({
            el: '#app',
            data: {
                content: 'hello world'
            }
        })
        setTimeout(function(){
            app.$data.content = 'bye world'
        },2000)
    </script>
</body>
</html>
```

#### 实现ToDoList

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <input type="text" v-model="inputValue"/>
        <button v-on:click="handleBtnClick">提交</button>
        <ul>
            <li v-for="item in list">{{item}}</li>
        </ul>
    </div>
    <script>
        var app = new Vue({
            el: '#app',
            data: {
                list: ['第一课','第二课'],
                inputValue: ''
            },
            methods: {
                handleBtnClick: function() {
                    this.list.push(this.inputValue)
                    this.inputValue = ''
                }
            }
        })
    </script>
</body>
</html>
```

#### 组件化：全局组件

自组件使用props接收父组件的值

父组件使用v-bind:向子组件传值

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="root">
        <input type="text" v-model="inputValue"/>
        <button @click="handleBtnClick">提交</button>
        <ul>
            <todo-item v-bind:content="item" v-for="item in list">{{item}}</todo-item> 
        </ul>
    </div>
    <script>
        Vue.component("TodoItem",{
            props: ['content'],
            template: "<li>{{content}}</li>"
        })
        var app = new Vue({
            el: '#root',
            data: {
                list: ['第一课','第二课'],
                inputValue: ''
            },
            methods: {
                handleBtnClick: function() {
                    this.list.push(this.inputValue)
                    this.inputValue = ''
                }
            }
        })
    </script>
</body>
</html>
```

#### 组件化：局部组件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="root">
        <input type="text" v-model="inputValue"/>
        <button @click="handleBtnClick">提交</button>
        <ul>
            <todo-item v-bind:content="item" v-for="item in list">{{item}}</todo-item>
        </ul>
    </div>
    <script>
        var TodoItem = {
            props: ['content'],
            template: "<li>{{content}}</li>"
        }
        var app = new Vue({
            el: '#root',
            components: {
                TodoItem: TodoItem
            },
            data: {
                list: ['第一课','第二课'],
                inputValue: ''
            },
            methods: {
                handleBtnClick: function() {
                    this.list.push(this.inputValue)
                    this.inputValue = ''
                }
            }
        })
    </script>
</body>
</html>
```

#### 简单的父子组件传值

在之前项目中新增功能：点击删除该项

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="root">
        <input type="text" v-model="inputValue"/>
        <button @click="handleBtnClick">提交</button>
        <ul>
            <todo-item :content="item"
                       v-bind:index="index"
                       v-for="(item,index) in list"
                       @delete="handleItemDelete">{{item}}</todo-item>
        </ul>
    </div>
    <script>
        var TodoItem = {
            props: ['content','index'],
            template: "<li @click='handleItemClick'>{{content}}</li>",
            methods: {
                handleItemClick: function() {
                    this.$emit("delete",this.index) 
                }
            }
        }
        var app = new Vue({
            el: '#root',
            components: {
                TodoItem: TodoItem
            },
            data: {
                list: ['第一课','第二课'],
                inputValue: ''
            },
            methods: {
                handleBtnClick: function() {
                    this.list.push(this.inputValue)
                    this.inputValue = ''
                },
                handleItemDelete: function(index) {
                    this.list.splice(index,1)
                }
            }
        })
    </script>
</body>
</html>
```

### Vue基础

#### 生命周期钩子

生命周期执行流程查看官网文档，一共有11个生命周期函数

```html
<body>
    <div id="root">hello world
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            beforeCreate: function() {
                console.log('before create')
            },
            created: function() {
                console.log('created')
            },
            beforeMount: function() {
                console.log(this.$el)
                console.log('beforeMount')
            },
            mounted: function() {
                console.log(this.$el)
                console.log('mounted')
            },
            beforeDestroy: function() {
                console.log("beforeDestroy")
            },
            destroyed: function() {
                console.log("destroyed") 
            },
            beforeUpdate: function() {
                console.log("beforeUpdate")
            },
            updated: function() {
                console.log("updated")
            },
        })
    </script>
</body>
```

#### 模版语法

```html
<body>
    <div id="root">
        {{name+' is awesome'}}
        <div v-text="name+' is awesome'"></div>
        <div v-html="name+' is awesome'"></div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                name: "<h1>Onion</h1>"
            } 
        })
    </script>
</body>
```

#### 计算属性

计算属性有缓存功能

```html
<body>
    <div id="root">
        {{fullName}}
        {{age}}
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                firstName: "Onion",
                lastName: "Handsome",
                age: 18
            },
            computed: {
                fullName: function() {
                    return this.firstName + " " + this.lastName
                }
            }
        })
    </script>
</body>
```

```html
<body>
    <div id="root">
        {{fullName()}}
        {{age}}
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                firstName: "Onion",
                lastName: "Handsome",
                age: 18
            },
            methods: {
                fullName: function() {
                    return this.firstName + " " + this.lastName
                }
            }
        })
    </script>
</body>
```

定义方法，不具有缓存功能，注意使用插值表达式需要加括号

####监听器

可以用watch实现上面相同的功能，也具有缓存功能

```html
<body>
    <div id="root">
        {{fullName}}
        {{age}}
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                firstName: "Onion",
                lastName: "Handsome",
                fullName: "Onion Handsome",
                age: 18
            },
            watch: {
                firstName: function() {
                    this.fullName = this.firstName + " " + this.lastName
                },
                lastName: function() {
                    this.fullName = this.firstName + " " + this.lastName 
                }
            }
        })
    </script>
</body>
```

#### setter与getter

```html
<body>
    <div id="root">
        {{fullName}}
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                firstName: "Onion",
                lastName: "Handsome",
            },
            computed: {
                fullName: {
                    get: function() {
                        return this.firstName + " " + this.lastName
                    },
                    set: function(value){
                        var arr = value.split(" ")
                        this.firstName = arr[0]
                        this.lastName = arr[1]
                        console.log(value)
                    }
                }
            }
        })
    </script>
</body>
```

在控制台输入vm.fullName = "Del Lee"查看效果

#### 样式绑定

实现功能：点击变红

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .activated {
            color: red
        }
    </style>
</head>
<body>
    <div id="root">
        <div @click="handleDivClick" :class="{activated:isActivated}">Hello world</div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                isActivated: false
            },
            methods: {
                handleDivClick: function() {
                    this.isActivated = !this.isActivated
                }
            }
        })
    </script>
</body>
</html>
```

写法二

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .activated {
            color: red
        }
    </style>
</head>
<body>
    <div id="root">
        <div @click="handleDivClick" :class="[activated]">Hello world</div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                activated: ""
            },
            methods: {
                handleDivClick: function() {
                    this.activated = (this.activated === "activated" ? "" : "activated")
                }
            }
        })
    </script>
</body>
</html>
```

写法三

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="root">
        <div :style="[styleObj,{fontSize:'20px'}]" @click="handleDivClick">Hello world</div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                styleObj: {
                    color: "black"
                }
            },
            methods: {
                handleDivClick: function() {
                    this.styleObj.color = (this.styleObj.color === "black" ? "red" : "black")
                }
            }
        })
    </script>
</body>
</html>
```

#### 条件渲染

v-if条件如果为false，则该dom不会存在。v-show只是隐藏。

还有v-else-if的用法，类比即可。

```html
<body>
    <div id="root">
        <div v-if="show">Hello world</div>
      	<viv v-else>Bye world</viv>
        <div v-show="show">Hello world</div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true,
            }
        })
    </script>
</body>
```

为何要指定Key

去掉key，在控制台输入vm.$data.show=true，input里面的内容不会清空，因为会复用dom

```html
<body>
    <div id="root">
        <div v-if="show">
            用户名: <input key="username"/>
        </div>
        <div v-else>
            邮箱名: <input key="email"/>
        </div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: false,
            }
        })
    </script>
</body>
```

#### 列表渲染

```html
<body>
    <div id="root">
        <template v-for="(item,index) of list" :key="item.id">
            <div>
                {{item.text}} --- {{index}}
            </div>
        </template>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                list: [{
                    id: "10175101201",
                    text: "onion"
                },{
                    id: "10175101202",
                    text: "mushroom"
                }]
            }
        })
    </script>
</body>
```

对象循环

```html
<body>
    <div id="root">
        <div v-for="(item,key,index) of userInfo">
            {{item}} --- {{key}} --- {{index}}
        </div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                userInfo: {
                    name: "Onion",
                    age: "18",
                    gender: "male"
                }
            }
        })
    </script>
</body>
```

Vue.set(vm.userInfo,"address":"beijing")

vm.$set(vm.userInfo,"address":"beijing")

对数组操作

Vue.set(vm.array,1,5) 将下标为1的数字改为5

### 组件

#### is

在子组件中，data必须为函数

```html
<body>
    <div id="root">
        <table>
            <tbody>
                <tr is="row"></tr>
            </tbody>
        </table>
    </div>
    <script>
        Vue.component('row',{
            data: function() {
                return {
                    content: 'this is content'
                }
            },
            template: '<tr><td>{{content}}</td></tr>'
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body>
```

不可写作

```json
data: {
		content: 'this is content'
}
```

#### ref

操作dom

```html
<body>
    <div id="root">
        <div ref='hello' @click="handleClick">hello world</div>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            methods: {
                handleClick: function(){
                    alert(this.$refs.hello.innerHTML)
                }
            }
        })
    </script>
</body>
```

作为组件引用

案例：计数器

```html
<body>
    <div id="root">
        <counter ref="one" @change="handleChange"></counter>
        <counter ref="two" @change="handleChange"></counter>
        <div>{{total}}</div>
    </div>
    <script>
        Vue.component('counter',{
            template: '<div @click="handleClick">{{number}}</div>',
            data: function() {
                return {
                    number: 0
                }
            },
            methods: {
                handleClick: function() {
                    this.number ++
                    this.$emit('change')
                }
            }
        })
        var vm = new Vue({
            el: "#root",
            data: {
                total: 0
            },
            methods: {
                handleChange: function() {
                    this.total = this.$refs.one.number + this.$refs.two.number
                }
            }
        })
    </script>
</body>
```

#### 父子组件数据传递

子组件不要直接修改父组件的值

```html
<body>
    <div id="root">
        <counter :count="3" @inc="handleIncrease"></counter>
        <counter :count="2" @inc="handleIncrease"></counter>
        <div>{{total}}</div>
    </div>
    <script>
        var counter = {
            props: ['count'],
            template: '<div @click="handleClick">{{number}}</div>',
            data: function() {
                return {
                    number: this.count
                }
            },
            methods: {
                handleClick: function() {
                    this.number ++
                    this.$emit('inc', 1)
                }
            }
        } 
        var vm = new Vue({
            el: "#root",
            data: {
                total: 5
            },
            components: {
                counter: counter
            },
            methods: {
                handleIncrease: function(step) {
                    this.total += step
                }
            }
        })
    </script>
</body>
```

#### 组件参数校验

```html
<body>
    <div id="root">
        <child content="hello world"></child>
    </div>
    <script>
        Vue.component('child',{
            props: {
                content: {
                    type: String,
                    required: false,
                    default: 'default value',
                    validator: function(value) {
                        return (value.length > 5)
                    }
                }
            },
            template: '<div>{{content}}</div>'
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body>
```

#### 非props特性

子组件不用props去接content，content会展示在dom标签中，此时使用{{content}}会报错

```html
<body>
    <div id="root">
        <child content="hello world"></child>
    </div>
    <script>
        Vue.component('child',{
            template: '<div>{{content}}</div>'
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body><body>
    <div id="root">
        <child @click="handleClick"></child>
    </div>
    <script>
        Vue.component('child',{
            template: '<div></div>'
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body>
```

#### 给组件绑定原生事件

```html
<body>
    <div id="root">
        <child @click.native="handleClick"></child>
    </div>
    <script>
        Vue.component('child',{
            template: '<div>Child</div>'
        })
        var vm = new Vue({
            el: "#root",
          	methods: {
            		handleClick: function(){
              		alert("click")
            		}
            }
        })
    </script>
</body>
```

#### 非父子组件传值

实现如下功能，点击一个content，另外一个content也会变成相同的内容。兄弟组件传值。

```html
<body>
    <div id="root">
        <child content="Onion"></child>
      	<child content="Mushroom"></child>
    </div>
    <script>
      	Vue.prototype.bus = new Vue()
        Vue.component('child',{
          	data: function(){
             		return {
                 		selfContent: this.content
                }
            },
          	props: {
              	content: String
            },
            template: '<div @click="handleClick">{{selfContent}}</div>',
          	methods: {
              	handleClick: function(){
                  	this.bus.$emit('change',this.selfContent)
                }
            },
          	mounted: function(){
              	var this_ = this
              	this.bus.$on('change', function(msg){
                		this_.selfContent = msg
                })
            }
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body>
```

#### 插槽

```html
<body>
    <div id="root">
        <child>
            <p>Onion</p>
        </child>
    </div>
    <script>
        Vue.component('child',{
            template: '<div><p>hello</p><slot>默认内容</slot></div>'
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body>
```

给插槽命名

```html
<body>
    <div id="root">
        <child>
            <div class="header" slot='header'>header</div>
            <div class="footer" slot='footer'>footer</div>
        </child>
    </div>
    <script>
        Vue.component('child',{
            template: `<div>
                        <slot name='header'>
                            <h1>default header</h1>
                        </slot>
                        <div>content</div><div>
                        <slot name='footer'>
                        </slot>
                        </div>`
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body>
```

#### 作用域插槽

```html
<body>
    <div id="root">
        <child>
           <template slot-scope="props">
               <h1>{{props.item}}</h1>
           </template>
        </child>
    </div>
    <script>
        Vue.component('child',{
            data: function() {
                return {
                    list: [1,2,3,4]
                }
            },
            template: `<div>
                        <ul>
                            <slot v-for="item of list" :item=item></slot>
                        </ul>
                        </div>`
        })
        var vm = new Vue({
            el: "#root"
        })
    </script>
</body>
```

#### 动态组件与v-once

```html
<body>
    <div id="root">
        <component :is="type"></component>
        <button @click="handleBtnClick">change</button>
    </div>
    <script>
        Vue.component('child-one',{
            template: `<div v-once>child-one</div>`
        })
        Vue.component('child-two',{
            template: `<div v-once>child-two</div>`
        })
        var vm = new Vue({
            el: "#root",
            data: {
                type: 'child-one'
            },
            methods: {
                handleBtnClick: function() {
                    this.type = this.type === 'child-one' ? 'child-two' : 'child-one'
                }
            }
        })
    </script>
</body>
```

### 动画

#### 过渡动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .fade-enter,
        .fade-leave-to {
            opacity: 0;
        }
        .fade-enter-active,
        .fade-leave-active {
            transition: opacity 3s;
        }
    </style>
</head>
<body>
    <div id="root">
        <transition name="fade">
            <div v-if="show">hello world</div>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
</html>
```

动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <link rel="stylesheet" href="http://www.bootcdn.cn/animate.css/">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        @keyframes bounce-in{
            0% {
                transform: scale(0)
            }
            50% {
                transform: scale(1.5)
            }
            100% {
                transform: scale(1)
            }
        }
        .active {
            transform-origin: left center;
            animation: bounce-in 1s;
        }
        .leave {
            transform-origin: left center;
            animation: bounce-in 1s reverse;
        }
    </style>
</head>
<body>
    <div id="root">
        <transition name="fade" enter-active-class="active" leave-active-class="leave">
            <div v-show="show">hello world</div>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
</html>
```

#### Animated CSS

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <link href="https://cdn.bootcss.com/animate.css/3.7.2/animate.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="root">
        <transition name="fade" enter-active-class="animated swing" leave-active-class="animated shake">
            <div v-show="show">hello world</div>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
</html>
```

#### 在Vue中使用过渡和动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <link href="https://cdn.bootcss.com/animate.css/3.7.2/animate.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .fade-enter, .fade-leave-to {
            opacity: 0;
        }
        .fade-enter-active, .fade-leave-active {
            transition: opacity 3s;
        }
    </style>
</head>
<body>
    <div id="root">
        <transition
        :duration="{enter:5000,leave:10000}"
        name="fade" 
        appear
        enter-active-class="animated swing fade-enter-active" 
        leave-active-class="animated shake fade-leave-active"
        appear-active-class="animated swing"
        >
            <div v-show="show">hello world</div>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
</html>
```

#### JS动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="root">
        <transition
            name="fade"
            @before-enter="handleBeforeEnter"
            @enter="handleEnter"
            @after-enter="handleAfterEnter"
        >
            <div v-show="show">hello world</div>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                },
                handleBeforeEnter: function(el) {
                    el.style.color = 'red'
                },
                handleEnter: function(el, done) {
                    setTimeout(()=>{
                        el.style.color = 'green'
                    },2000)
                    setTimeout(()=>{
                        done()
                    },4000)
                },
                handleAfterEnter: function(el) {
                    el.style.color = '#000'
                }
            }
        })
    </script>
</body>
</html>
```

#### 引入velocity

```html
<body>
    <div id="root">
        <transition
            name="fade"
            @before-enter="handleBeforeEnter"
            @enter="handleEnter"
            @after-enter="handleAfterEnter"
        >
            <div v-show="show">hello world</div>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                },
                handleBeforeEnter: function(el) {
                    el.style.opacity = 0;
                },
                handleEnter: function(el, done) {
                    Velocity(el, {opacity: 1}, {duration:1000,complete:done})
                },
                handleAfterEnter: function(el) {
                    console.log('end')
                }
            }
        })
    </script>
</body>
```

#### 多组件动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .v-enter, .v-leave-to {
            opacity: 0;
        }
        .v-enter-active, .v-leave-active {
            transition: opacity 1s;
        }
    </style>
</head>
<body>
    <div id="root">
        <transition mode="in-out">
            <div v-if="show" key="hello">hello world</div>
            <div v-else key="bye">bye world</div>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
</html>
```

与动态组件结合使用

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .v-enter, .v-leave-to {
            opacity: 0;
        }
        .v-enter-active, .v-leave-active {
            transition: opacity 1s;
        }
    </style>
</head>
<body>
    <div id="root">
        <transition mode="out-in">
            <component :is="type"></component>
        </transition>
        <button @click="handleBtnClick">切换</button>
    </div>
    <script>
        Vue.component('child',{
            template: '<div>child</div>'
        })
        Vue.component('child-one',{
            template: '<div>child-one</div>'
        })
        var vm = new Vue({
            el: "#root",
            data: {
                type: 'child'
            },
            methods: {
                handleBtnClick: function() {
                    this.type = this.type === 'child' ? 'child-one' : 'child'
                }
            }
        })
    </script>
</body>
</html>
```

#### 列表过渡

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hello world</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .v-enter, .v-leave-to {
            opacity: 0;
        }
        .v-enter-active, .v-leave-active {
            transition: opacity 1s;
        }
    </style>
</head>
<body>
    <div id="root">
        <transition-group>
            <div v-for="item of list" :key="item.id">
                {{item.title}}
            </div>
        </transition-group>
        <button @click="handleBtnClick">add</button>
    </div>
    <script>
        var count = 0;
        var vm = new Vue({
            el: "#root",
            data: {
                list: []
            },
            methods: {
                handleBtnClick: function() {
                    this.list.push({
                        id: count++,
                        title: 'hello world'
                    })
                }
            }
        })
    </script>
</body>
</html>
```

#### 封装

```html
<body>
    <div id="root">
        <fade :show="show">
            <div>hello world</div>
        </fade>
        <button @click="handleBtnClick">toggle</button>
    </div>
    <script>
        Vue.component('fade', {
            props: ['show'],
            template: `<transition @before-enter="handleBeforeEnter" @enter="handleEnter"><slot v-if="show"></slot></transition>`,
            methods: {
                handleBeforeEnter: function(el) {
                    el.style.color = 'red'
                },
                handleEnter: function(el, done) {
                    setTimeout(()=>{
                        el.style.color = 'green'
                        done()
                    },2000)
                },
            }
        })
        var count = 0;
        var vm = new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleBtnClick: function() {
                    this.show = !this.show
                }
            }
        })
    </script>
</body>
```

### 项目实战

#### 项目初始化

npm install --global vue-cli

vue init webpack Travel

#### 使用Axios发送Ajax请求

因项目比较复杂，只提取局部代码示例

使用activated钩子函数提高性能，不重复发送请求

```vue
<script>
import axios from 'axios'
export default {
  name: 'Home',
  data () {
    return {
      lastCity: '',
      swiperList: [],
      iconList: [],
      recommendList: [],
      weekendList: []
    }
  },
  methods: {
    getHomeInfo () {
      axios.get('/api/index.json?city=' + this.city)
        .then(this.getHomeInfoSucc)
    },
    getHomeInfoSucc (res) {
      res = res.data
      if (res.ret && res.data) {
        const data = res.data
        this.swiperList = data.swiperList
        this.iconList = data.iconList
        this.recommendList = data.recommendList
        this.weekendList = data.weekendList
      }
    }
  },
  mounted () {
    this.lastCity = this.city
    this.getHomeInfo()
  },
  activated () {
    if (this.lastCity !== this.city) {
      this.lastCity = this.city
      this.getHomeInfo()
    }
  }
}
</script>
```

axios请求封装

```javascript
import axios from 'axios'

let base = 'http://localhost/api/';
export const postRequest = (url, data, params) => {
  return axios({
    method: 'post',
    url: `${base}${url}`,
    data: data,
    params: params,
    transformRequest: [function (data) {
      data = JSON.stringify(data);
      return data
    }],
    headers: {
      'Content-Type': 'application/json',
      'token': sessionStorage.getItem("token"),
    }
  });
};
export const putRequest = (url, data, params) => {
  return axios({
    method: 'put',
    url: `${base}${url}`,
    data: data,
    params: params,
    transformRequest: [function (data) {
      data = JSON.stringify(data);
      return data;
    }],
    headers: {
      'Content-Type': 'application/json',
      'token': sessionStorage.getItem("token"),
    }
  });
};
export const getRequest = (url,params) => {
  return axios({
    method: 'get',
    url: `${base}${url}`,
    params: params,
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      'token': sessionStorage.getItem("token"),
    }
  });
};

```

```javascript
import axios from 'axios'

let base = '';
export const postRequest = (url, params) => {
  return axios({
    method: 'post',
    url: `${base}${url}`,
    data: params,
    transformRequest: [function (data) {
      // Do whatever you want to transform the data
      let ret = ''
      for (let it in data) {
        ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
      }
      return ret
    }],
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  });
}
export const uploadFileRequest = (url, params) => {
  return axios({
    method: 'post',
    url: `${base}${url}`,
    data: params,
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  });
}
export const putRequest = (url, params) => {
  return axios({
    method: 'put',
    url: `${base}${url}`,
    data: params,
    transformRequest: [function (data) {
      let ret = ''
      for (let it in data) {
        ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
      }
      return ret
    }],
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  });
}
export const deleteRequest = (url) => {
  return axios({
    method: 'delete',
    url: `${base}${url}`
  });
}
export const getRequest = (url,params) => {
  return axios({
    method: 'get',
    data:params,
    transformRequest: [function (data) {
      let ret = ''
      for (let it in data) {
        ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
      }
      return ret
    }],
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    url: `${base}${url}`
  });
}

```

#### 使用Route-Link

可以添加参数，使用this.$route.params.id获取

```vue
<router-link to='/city'>
      <div class="header-right">
        {{this.city}}
      </div>
</router-link>

<router-link
        v-for="item of list"
        :key="item.id"
        :to="'/detail/' + item.id">
</router-link>
```

在router/index.js下定义

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import Home from '@/pages/home/Home'
import City from '@/pages/city/City'
import Detail from '@/pages/detail/Detail'

Vue.use(Router)

export default new Router({
  routes: [{
    path: '/',
    name: 'Home',
    component: Home
  }, {
    path: '/city',
    name: 'City',
    component: City
  }, {
    path: '/detail/:id',
    name: 'Detail',
    component: Detail
  }],
  scrollBehavior (to, from, savedPosition) {
    return { x: 0, y: 0 }
  }
})
```

app.vue定义

keep-alive的用法查阅官方文档

```vue
<template>
  <div id="app">
    <keep-alive exclude="Detail">
      <router-view/>
    </keep-alive>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style></style>
```

#### 对全局事件解绑

```javascript
mounted () {
    window.addEventListener('scroll', this.handleScroll)
},
unmounted () {
    window.removeEventListener('scroll', this.handleScroll)
}
```

#### 递归组件

递归组件可以用于树形评论等结构中

在style中设置样式，实现子组件的格式缩进

```vue
<template>
  <div>
    <div
      class="item"
      v-for="(item, index) of list"
      :key="index"
    >
      <div class="item-title border-bottom">
        <span class="item-title-icon"></span>
        {{item.title}}
      </div>
      <div v-if="item.children" class="item-chilren">
        <detail-list :list="item.children"></detail-list>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DetailList',
  props: {
    list: Array
  }
}
</script>

<style lang="stylus" scoped>
  .item-title-icon
    position: relative
    left: .06rem
    top: .06rem
    display: inline-block
    width: .36rem
    height: .36rem
    background: url(http://s.qunarzz.com/piao/image/touch/sight/detail.png) 0 -.45rem no-repeat
    margin-right: .1rem
    background-size: .4rem 3rem
  .item-title
    line-height: .8rem
    font-size: .32rem
    padding: 0 .2rem
  .item-chilren
    padding: 0 .2rem
</style>

```

#### 使用LocalStorage、SessionStorage

sessionStorage.setItem("key","value")

#### 使用Vuex

在store目录下创建index.js，state.js，mutation.js

index.js

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import state from './state'
import mutations from './mutations'

Vue.use(Vuex)

export default new Vuex.Store({
  state,
  mutations
})

```

mutation.js

```javascript
export default {
  changeCity (state, city) {
    state.city = city
    try {
      localStorage.city = city
    } catch (e) {}
  }
}
```

state.js

```javascript
let defaultCity = '上海'
try {
  if (localStorage.city) {
    defaultCity = localStorage.city
  }
} catch (e) {}

export default {
  city: defaultCity
}
```

#### 使用Vue Router

实现功能：点击选择城市后跳转回到首页

```vue
<script>
import { mapState, mapMutations } from 'vuex'
export default {
  name: 'CityList',
  computed: {
    ...mapState({
      currentCity: 'city'
    })
  },
  methods: {
    handleCityClick (city) {
      this.changeCity(city)
      this.$router.push('/')
    },
    ...mapMutations(['changeCity'])
  }
}
</script>
```

#### 过滤器

```javascript
import Vue from 'vue'
Vue.filter("formatDate", function formatDate(value) {
  var date = new Date(value);
  var year = date.getFullYear();
  var month = date.getMonth() + 1;
  var day = date.getDate();
  if (month < 10) {
    month = "0" + month;
  }
  if (day < 10) {
    day = "0" + day;
  }
  return year + "-" + month + "-" + day;
});
Vue.filter("formatDateTime", function formatDateTime(value) {
  var date = new Date(value);
  var year = date.getFullYear();
  var month = date.getMonth() + 1;
  var day = date.getDate();
  var hours = date.getHours();
  var minutes = date.getMinutes();
  if (month < 10) {
    month = "0" + month;
  }
  if (day < 10) {
    day = "0" + day;
  }
  return year + "-" + month + "-" + day + " " + hours + ":" + minutes;
});

```

#### 路径转发

```javascript
proxyTable: {
		'/api': {
        target: 'http://localhost:8080/',
        changeOrigin: true,
        pathRewrite: {
          '^/api': '',
        }
		}
}
```


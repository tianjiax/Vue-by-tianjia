# Vue-by-tianjia
对vue官网文档的一些个人总结，vue开发中遇到的一些问题及解决方法记录。
## 表单输入绑定
### 修饰符
##### .lazy
在默认情况下，v-model 在每次 input 事件触发后将输入框的值与数据进行同步 (除了上述输入法组合文字时)。你可以添加 lazy 修饰符，从而转变为使用 change 事件进行同步：
```html
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" >
```
##### .number
如果想自动将用户的输入值转为数值类型，可以给 v-model 添加 number 修饰符：
```html
<input v-model.number="age" type="number">
```
这通常很有用，因为即使在 type="number" 时，HTML 输入元素的值也总会返回字符串。

##### .trim
如果要自动过滤用户输入的首尾空白字符，可以给 v-model 添加 trim 修饰符：
```html
<input v-model.trim="msg">
```

### 值绑定
> 绑定传递的值，并非只能是false/ture/value

##### 复选框值
```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```
```js
// 当选中时
vm.toggle === 'yes'
// 当没有选中时
vm.toggle === 'no'
```
> 这里的 true-value 和 false-value 特性并不会影响输入控件的 value 特性，因为浏览器在提交表单时并不会包含未被选中的复选框。如果要确保表单中这两个值中的一个能够被提交，(比如“yes”或“no”)，请换用单选按钮。

##### 单选按钮
```html
<input type="radio" v-model="pick" v-bind:value="a">
```
```js
// 当选中时
vm.pick === vm.a
```

##### 选择框的选项
```html
<select v-model="selected">
    <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```
```js
// 当选中时
typeof vm.selected // => 'object'
vm.selected.number // => 123
```
## 事件处理
### 为什么在 HTML 中监听事件
你可能注意到这种事件监听的方式违背了关注点分离 (separation of concern) 这个长期以来的优良传统。但不必担心，因为所有的 Vue.js 事件处理方法和表达式都严格绑定在当前视图的 ViewModel 上，它不会导致任何维护上的困难。实际上，使用 v-on 有几个好处：

1. 扫一眼 HTML 模板便能轻松定位在 JavaScript 代码里对应的方法。
1. 因为你无须在 JavaScript 里手动绑定事件，你的 ViewModel 代码可以是非常纯粹的逻辑，和 DOM 完全解耦，更易于测试。
1. 当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除。你无须担心如何自己清理它们。

### 按键修饰符
在监听键盘事件时，我们经常需要检查常见的键值。Vue 允许为 v-on 在监听键盘事件时添加按键修饰符：
```html
<!-- 只有在 `keyCode` 是 13 时调用 `vm.submit()` -->
<input v-on:keyup.13="submit">
<!-- 同上 -->
<input v-on:keyup.enter="submit">

<!-- 缩写语法 -->
<input @keyup.enter="submit">
```
全部的按键别名：

- .enter
- .tab
- .delete (捕获“删除”和“退格”键)
- .esc
- .space
- .up
- .down
- .left
- .right

可以通过全局 config.keyCodes 对象自定义按键修饰符别名：
```js
// 可以使用 `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```
##### 自动匹配按键修饰符
> 2.5.0 新增

你也可直接将 KeyboardEvent.key 暴露的任意有效按键名转换为 kebab-case 来作为修饰符：
```html
<input @keyup.page-down="onPageDown">
```
在上面的例子中，处理函数仅在 $event.key === 'PageDown' 时被调用。

> 有一些按键 (.esc 以及所有的方向键) 在 IE9 中有不同的 key 值, 如果你想支持 IE9，它们的内置别名应该是首选。

### 系统修饰键
按下相应按键时才触发鼠标或键盘事件的监听器。
- .ctrl
- .alt
- .shift
- .meta

```html
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```
> 注意：在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。在 Sun 操作系统键盘上，meta 对应实心宝石键 (◆)。在其他特定键盘上，尤其在 MIT 和 Lisp 机器的键盘、以及其后继产品，比如 Knight 键盘、space-cadet 键盘，meta 被标记为“META”。在 Symbolics 键盘上，meta 被标记为“META”或者“Meta”。

##### .exact 修饰符
> 2.5.0 新增

.exact 修饰符允许你控制由精确的系统修饰符组合触发的事件。
```html
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

##### 鼠标按钮修饰符
> 2.2.0 新增

- .left
- .right
- .middle

这些修饰符会限制处理函数仅响应特定的鼠标按钮。

### 事件修饰符
在事件处理程序中调用 event.preventDefault() 或 event.stopPropagation() 是非常常见的需求。尽管我们可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。

为了解决这个问题，Vue.js 为 v-on 提供了事件修饰符。之前提过，修饰符是由点开头的指令后缀来表示的。

- .stop
- .prevent
- .capture
- .self
- .once

```html
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>
```
> 使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用
==v-on:click.prevent.self==
会阻止所有的点击，而
==v-on:click.self.prevent==
只会阻止对元素自身的点击。


> 2.1.4 新增

```html
<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="doThis"></a>
```
不像其它只能对原生的 DOM 事件起作用的修饰符，.once 修饰符还能被用到自定义的组件事件上。如果你还没有阅读关于组件的文档，现在大可不必担心。

> 2.3.0 新增

Vue 还对应 addEventListener 中的 passive 选项提供了 .passive 修饰符。

```html
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<!-- 而不会等待 `onScroll` 完成  -->
<!-- 这其中包含 `event.preventDefault()` 的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```
这个 .passive 修饰符尤其能够提升移动端的性能。

> 不要把 .passive 和 .prevent 一起使用，因为 .prevent 将会被忽略，同时浏览器可能会向你展示一个警告。请记住，.passive 会告诉浏览器你_不_想阻止事件的默认行为。

## 列表渲染
### 一个组件的 v-for
> 了解组件相关知识，查看 组件。完全可以先跳过它，以后再回来查看。

在自定义组件里，你可以像任何普通元素一样用 ==v-for== 。
```html
<my-component v-for="item in items" :key="item.id"></my-component>
```
> 2.2.0+ 的版本里，当在组件中使用 ==v-for== 时，==key== 现在是必须的。

然而，任何数据都不会被自动传递到组件里，因为组件有自己独立的作用域。为了把迭代数据传递到组件里，我们要用 ==props== ：
```html
<my-component
  v-for="(item, index) in items"
  v-bind:item="item"
  v-bind:index="index"
  v-bind:key="item.id"
></my-component>
```
不自动将 item 注入到组件里的原因是，这会使得组件与 v-for 的运作紧密耦合。明确组件数据的来源能够使组件在其他场合重复使用。

下面是一个简单的 todo list 的完整例子：
```html
<div id="todo-list-example">
  <input
    v-model="newTodoText"
    v-on:keyup.enter="addNewTodo"
    placeholder="Add a todo"
  >
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:key="todo.id"
      v-bind:title="todo.title"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
```
注意这里的 ==is="todo-item"== 属性。这种做法在使用 DOM 模板时是十分必要的，因为在 ==\<ul>== 元素内只有 ==\<li>== 元素会被看作有效内容。这样做实现的效果与 ==\<todo-item>== 相同，但是可以避开一些潜在的浏览器解析错误。查看 DOM 模板解析说明 来了解更多信息。
```js
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">X</button>\
    </li>\
  ',
  props: ['title']
})

new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      {
        id: 1,
        title: 'Do the dishes',
      },
      {
        id: 2,
        title: 'Take out the trash',
      },
      {
        id: 3,
        title: 'Mow the lawn'
      }
    ],
    nextTodoId: 4
  },
  methods: {
    addNewTodo: function () {
      this.todos.push({
        id: this.nextTodoId++,
        title: this.newTodoText
      })
      this.newTodoText = ''
    }
  }
})
```

### 动态添加对象
还是由于 JavaScript 的限制，Vue 不能检测对象属性的添加或删除：
```js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 现在是响应式的

vm.b = 2
// `vm.b` 不是响应式的
```
对于已经创建的实例，Vue 不能动态添加根级别的响应式属性。但是，可以使用 Vue.set(object, key, value) 方法向嵌套对象添加响应式属性。例如，对于：
```js
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})
```
你可以添加一个新的 age 属性到嵌套的 userProfile 对象：
```js
Vue.set(vm.userProfile, 'age', 27)
```
你还可以使用 vm.$set 实例方法，它只是全局 Vue.set 的别名：
```js
vm.$set(vm.userProfile, 'age', 27)
```
有时你可能需要为已有对象赋予多个新属性，比如使用 Object.assign() 或 _.extend()。在这种情况下，你应该用两个对象的属性创建一个新的对象。所以，如果你想添加新的响应式属性，不要像这样：
```js
Object.assign(vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
你应该这样做：
```js
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

### key
当 Vue.js 用 v-for 正在更新已渲染过的元素列表时，它默认用“就地复用”策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。这个类似 Vue 1.x 的 track-by="$index" 。

这个默认的模式是高效的，但是只适用于不依赖子组件状态或临时 DOM 状态 (例如：表单输入值) 的列表渲染输出。

为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，你需要为每项提供一个唯一 key 属性。理想的 key 值是每项都有的且唯一的 id。这个特殊的属性相当于 Vue 1.x 的 track-by ，但它的工作方式类似于一个属性，所以你需要用 v-bind 来绑定动态值 (在这里使用简写)：

```html
<div v-for="item in items" :key="item.id">
  <!-- 内容 -->
</div>
```

建议尽可能在使用 v-for 时提供 key，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升。

因为它是 Vue 识别节点的一个通用机制，key 并不与 v-for 特别关联，key 还具有其他用途，我们将在后面的指南中看到其他用途。
### 一个对象的 v-for
你也可以用 v-for 通过一个对象的属性来迭代。
```html
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }}: {{ value }}
</div>
```
```js
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      firstName: 'John',
      lastName: 'Doe',
      age: 30
    }
  }
})
```
```
// 输出
0. firstName: John
1. lastName: Doe
2. age: 30
```
## 条件渲染
### v-if 与 v-show
```html
<div v-if="false"></div> // 直接不渲染
<div v-show="false"></div> // style="display:none"
```
==v-if== 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

==v-if== 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

相比之下，==v-show== 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

一般来说，==v-if== 有更高的切换开销，而 ==v-show== 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 ==v-show== 较好；如果在运行时条件很少改变，则使用 ==v-if== 较好

## 模版语法
### 使用 JavaScript 表达式
迄今为止，在我们的模板中，我们一直都只绑定简单的属性键值。但实际上，对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```
这些表达式会在所属 Vue 实例的数据作用域下作为 JavaScript 被解析。有个限制就是，每个绑定都只能包含单个表达式，所以下面的例子都不会生效。

```html
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```
> 模板表达式都被放在沙盒中，只能访问全局变量的一个白名单，如 Math 和 Date 。你不应该在模板表达式中试图访问用户定义的全局变量。

## 其他
### 不要在选项属性或回调上使用箭头函数
> 不要在选项属性或回调上使用箭头函数，比如 created: () => console.log(this.a) 或 vm.$watch('a', newValue => this.myMethod())。因为箭头函数是和父级上下文绑定在一起的，this 不会是如你所预期的 Vue 实例，经常导致 Uncaught TypeError: Cannot read property of undefined 或 Uncaught TypeError: this.myMethod is not a function 之类的错误。

### $获取实例及方法
> 除了数据属性，Vue 实例还暴露了一些有用的实例属性与方法。它们都有前缀 $，以便与用户定义的属性区分开来。

```js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```
### 阻止动态渲染
> Object.freeze()，这会阻止修改现有的属性，也意味着响应系统无法再_追踪_变化。

```js
// 括号里面为vue的数据名字
Object.freeze(vm.a)
```

### vuejs devtools
> vue-devtools是一款基于chrome游览器的插件，用于调试vue应用，这可以极大地提高我们的调试效率。
- https://www.cnblogs.com/momozjm/p/7098476.html
- https://segmentfault.com/a/1190000009682735

### [字符串与数组转化](http://www.jb51.net/article/78550.htm)

### #v-cloak避免闪烁
> 这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 [v-cloak] { display: none } 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。
```
[v-cloak] {
display: none;
} 
<div v-cloak>
{{ message }}
</div>
```
> 带有v-clock的的元素设置为display:none，隐藏掉，在等到vue解析到带有v-clock的节点时候，会把attribute和class同时remove掉，这样就可以实现防止节点的闪烁。

### IE11不兼容的处理
> IE11的话不兼容methods里面的getData(transmit){}这种写法，如若需要兼容改写为getData:function(transmit){}

### watch 的问题
> watch的判定监听先判断类型，例如：
```
"list.temp5":{
	handler:function(val,oldval){  
        console.log(val)  
    },  
    deep:true//对象内部的属性监听，也叫深度监听  
}
```
> 如果只是改部分字段不改类型的话，可能会导致无法监听。例如数组，我们可以先定义一个临时的字段进行存储，然后对原来的数组进行一个置空，再对修改的值进行索引填充再返回到数组内。

### 跨页面iframe修改另外一个vue的字段
> 普通的可以按以下的 “ iframe 子页面调取父页面方法”来进行操作，如果是数组的话，必须用到上面的watch处理方法。不然数据无法进行渲染，切记。

### Ajax JSON数据格式传数组

```js
data:JSON.stringify(Obtain),
contentType: "application/json;charset=utf-8",
```


### 子组件调取父组件方法
```js
this.$parent.pageDate.pageCuur = obj.curr;
this.$parent.returnList(obj.curr);
```

### 父组件调取子组件方法
> 尽管有 prop 和事件，但是有时仍然需要在 JavaScript 中直接访问子组件。为此可以使用 ref 为子组件指定一个引用 ID。当 ref 和 v-for 一起使用时，获取到的引用会是一个数组，包含和循环数据源对应的子组件。

###### 父文件
```html
<page ref = 'pageref'></page>   
```
```js
this.$refs.pageref.getPage();
```
##### 子组件
```js
// 定义组件
var pageTemplate = {
    template: '<div id="page"></div>',
    data(){
      return {
          
      }
    },
    methods:{
        getPage:function(){
            
        }
    },

};
```

### iframe 子页面调取父页面方法
> 用 top.Vue实例.方法 或  window.top.Vue实例.方法 或 $(Dom, top.document)[0].contentWindow.supplies

### 保留字
> vue里面的methods方法不能会delete，因为delete为保留字。

### 监控输入框及下拉框变化
```html
<!--输入框-->
<input type="text" id="cardsNum2"  value="1" v-on:input ="inputFunc">
<!--下拉框-->
<select name="" v-model="basic.a10" id="" v-on:change="validateA10()">
    <option value="">请选择</option>
    <option value="1">选项1</option>
    <option value="2">选项2</option>
    <option value="3">选项3</option>
</select>
```

### v-bind和v-on
> v-bind和v-on可以简写为 : 和 @

### class类的出现与否
> 第一个值为你的类名，第二个值为vue绑定的值，true或者false。

```html
<div :class="{previewc :　preview}">
```
 
### 防止出现生产提示
![image](http://img.blog.csdn.net/20170502094828428?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTU1MzQ3ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
```js
Vue.config.productionTip = false
```

### *.vue文件
> *.vue 文件，是一个自定义的文件类型，用类 HTML 语法描述一个 Vue 组件。每个 .vue文件包含三种类型的顶级语言块 <template>, <script> 和 <style>。这三个部分分别代表了 html,js,css。

### vue路由

> vue.config.productiontip = false  设置为 false 以阻止 vue 在启动时生成生产提示。

### vue项目

> vue 对绑定实例的元素内可以进行vue的操作，但是需要注意，不要嵌套绑定。

```js
var Vue = new Vue({
    el: '#vue',// vue绑定元素
    data:{
       id: "",
       ...
    },// 数据渲染
    methods: {},// 方法定义
    mounted() {}, // 方法调用
    computed: {}, // 计算属性
    watch:{}// 监控值的变化
})
```
> vue data里面的数据可以实时更新，利用v-model来进行输出

### 其它注意点
> 通过vue来获取到元素，对DOM的操作与js/jq相同

```js
$(e.currentTarget)
```
> 渲染让self等于vue的this，里面（self.方法/数据）调用方法及数据

```js
var self = this
```
> vue内部使用vue的名称.方法/数据调用方法及数据

> 使用 mounted 可以调用 methods 中的方法或者数据，例子：

```js
mounted: function () {
  this.refresh();
  this.data
}
```

> 外部调用vue的方法或者数据，利用vue实例名加对应的方法或者数据即可，例子：

```js
var Vue = new Vue({})

Vue.fun();
vue.data
```

> 计算属性，对数据进行实时计算，调用listInfo可以用reversedListInfo替代

```js
computed: {
// 计算属性
	reversedListInfo: function () {
	  // `this` 指向 vm 实例
	  return this.listInfo
	}
}
```
> vue方法来传值进来，类似js函数

```js
checkBox(index,e,ary)
```
> 添加json结构不要var，直接整个添加进去就好，不然指向会出问题

> 数据加载完成时候才调用

```js
self.$nextTick(function(){})
```
> 打开的窗口.iframe窗口.vue实例.方法（调用iframe的vue的方法）

```js
editForm.frameWindow.registerPage.stop();
```



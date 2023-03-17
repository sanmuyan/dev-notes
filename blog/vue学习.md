# Vue 学习

## 创建实例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
    <div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        // 数据区
        data() {
            return {
                message: 'hello world',
            }
        },
        // 事件处理，页面重新渲染是执行
        methods: {

        },
        // 复杂事件处理，依赖属性发生变化时执行
        computed: {

        },
        // 事件异步处理，监听的属性变化时执行
        watch: {

        },
        // 模板内容
        template: `
            <div>{{message}}</div>
            <demo/>
        `
    })
    // 子组件
    app.component('demo', {
        template: 
        `
            <div>我是子组件</div>
        `
    })
    // 组件绑定
    const vm = app.mount('#root')

</script>
</html>

```

## 生命周期函数

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
    <div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                message: 'hello world'
            }
        },
        // 在实例生成之前会自动执行的函数
        beforeCreate() {
          console.log('beforeCreate')
        },
        // 在实例生成之后会自动执行的函数
        created() {
            console.log('created')
        },
        // 在组件内容被渲染到页面之前自动执行的函数
        beforeMount() {
            console.log(document.getElementById('root').innerHTML,'beforeMount')
        },
        // 在组件内容被渲染到页面之后自动执行的函数
        mounted() {
            console.log(document.getElementById('root').innerHTML,'mounted')
        },
        // 当数据变化时自动执行的函数
        beforeUpdate(){
          console.log('beforeUpdate')
        },
        // 当数据变化同时页面完成更新后时自动执行的函数
        updated(){
            console.log('updated')
        },
        // 当vue 应用失效后执行的函数
        beforeUnmount(){
            console.log('beforeUnmount')
        },
        // 当vue 应用失效后，同时dom销毁后自动执行的函数
        unmounted(){
            console.log('unmounted')
        },
        // 如果有 template 则渲染 template 内容，如果没有则直接渲染id=root的容器
        template: "<div>{{message}}</div>"
    })
    const vm = app.mount('#root')

</script>
</html>
```

## 指令

### 常用指令

```html
<!-- 文本转成 html         -->
<div v-html="message"></div>

<!-- 属性标题         -->
<div title="message">hello world</div>

<!-- 属性绑定变量    v-bind 简写 :     -->
<div v-bind="message"></div>
<div :="message"></div>

<!-- 控制能否输入         -->
<input v-bind:disabled="disable" />

<!-- 属性不会改变         -->
<div v-once>{{message}}</div>

<!-- 判断div标签是否展示 v-if 会销毁dom v-show 不会销毁dom        -->
<div v-if="isShow">{{message}}</div>
<div v-show="isShow">{{message}}</div>

<!-- 点击事件 执行 handleClick 方法 v-on: 简写 @       -->
<div v-on:click="handleClick">{{message}}</div>
<div @click="handleClick">{{message}}</div>

<!-- 动态绑定        -->
<div @[event]="handleClick">{{message}}</div>
<div :[name]="message">{{message}}</div>

<!-- 双向绑定       -->
<div>
    <input v-model="message">
</div>

```

### 修饰符

```html
<!-- 事件修饰符 click.prevent 阻止默认行为       -->
<form action="https://www.baidu.com" v-on:click.prevent="handleClick">
    <button type="submit">提交</button>
</form>

<!-- 事件修饰符 click.stop 阻止事件继续冒泡       -->
<div v-on:click="handleAlert">
    {{message}} <p></p>
    <button v-on:click.stop="">按钮</button>
</div>

<!-- 按键修饰符 按回车后执行 handleAlert       -->
<div>
    <input v-on:keydown.enter="handleAlert">
</div>

<!-- 按键修饰符 按回车后执行 handleAlert       -->
<div>
    <input v-on:keydown.enter="handleAlert">
</div>

<!-- 按键修饰符 按住ctrl键点击执行 handleAlert       -->
<div>
    <button v-on:keydown.ctrl.exact="handleAlert">按住ctrl键</button>
</div>
```

## 计算|方法|监听

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
    // 能用 computed 和 methods 实现的，优先用 computed 实现 ，因为有缓存
    // 能用 computed 和 watch 实现的，优先用 computed 实现 ，因为实现起来简单        
        data() {
            return {
                message: 'hello world',
                count: 2,
                price: 5
            }
        },
        methods: {
            getTotal() {
                return Date.now()
            }
        },
        computed: {
            getTotal2() {
                return Date.now()
            }
        },
        watch: {
            // price 发现变化是执行
            price(current, prev) {
                console.log('price 现在的值=', current, 'price 上个值=', prev)
                setTimeout(() => {
                    console.log('price changed')
                }, 3000)
            }
        },
        template: `
          <div>{{ getTotal }}</div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

## 样式

### 样式绑定

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
<style>
    .red {
        color: red;
    }

    .green {
        color: green;
    }
</style>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                classString: 'red',
                classObject: {green: true},
                classArray: ['red', 'green'],
                styleObject: {color: 'orange'}
            }
        },
        // v-bind:class 绑定 style 标签样式
        // v-bind:style 直接在行内绑定样式
        template: `
          <div v-bind:class="classString">Hello World</div>
          <div v-bind:class="classObject">Hello World</div>
          <div v-bind:class="classArray">Hello World</div>
          <div v-bind:style="styleObject">Hello World</div>
          <demo class="green"/>
        `
    })
    app.component('demo', {
        // v-bind:class="$attrs.class 表示绑定父组件class
        template:
            `
          <div v-bind:class="$attrs.class">子组件1</div>
          <div class="red">子组件2</div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

### 列表循环渲染

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                listObject: {
                    Age: 18
                }
            }
        },
        methods: {
            handleAddBtnClick() {
                this.listObject.Age = this.listObject.Age + 1
            }

        },
        template: `
          <div>
            <div v-for="(v, k) in listObject">
              <div v-if="v  > '18'">
                {{ k }} -- {{ v }}
              </div>
            </div>
            <div>
              <button v-on:click="handleAddBtnClick">加一岁</button>
            </div>
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

## 数据双向绑定

### 勾选

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                message: []
            }
        },
        template: `
          <div>
          你输入了：{{message}}
            <p></p>}
            a <input type="checkbox" v-model="message" value="a">
            b <input type="checkbox" v-model="message" value="b">
            c <input type="checkbox" v-model="message" value="c">
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                message: false,
            }
        },
        template: `
          <div>
          你选择了：{{ message }}
          <p></p>
          <input type="checkbox" v-model="message" true-value="hello" false-value="world">
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

### 单选

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                message: ''
            }
        },
        template: `
          <div>
          你选择了：{{message}}
            <p></p>}
            a <input type="radio" v-model="message" value="a">
            b <input type="radio" v-model="message" value="b">
            c <input type="radio" v-model="message" value="c">
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>

```

### 列表

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                message: '',
                // 多选 message: []
                options: [{
                        text: 'A', value: 'A',
                    },
                    {
                        text: 'B', value: 'B',
                    },
                    {
                        text: 'C', value: 'C',
                    },

                ]
            }
        },
        template: `
          <div>
          你选择了：{{ message }}
          <p></p>
          <!--    多选 <select v-model="message" multiple>-->
          <select v-model="message">-->
            <option disabled value="">请选择</option>
            <option v-for="item in options">{{item.text}}</option>
          </select>
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

## 组件

### 组件通信

#### 传参

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    // 局部组件，不使用不会占用资源
    const Counter = ({
        template: `<div>局部组件</div>`
        }
    )
    const app = Vue.createApp({
        // 注册局部组件
        components: {counter2: Counter},
        data() {
            return {
                message: "父组件传参",
            }
        },
        //  <div><counter v-bind:content="message"/></div>
        template: `
          <div>
            <counter content="message"/>
          </div>
          <div>
            <counter2/>
          </div>
        `
    })
    // 全局子组件，即使不使用也会占用资源
    // 子组件使能使用父组件传递的数据但不能修改数据
    // 在不定义 props 接收数据时，可以用 v-bind:$attrs this.$attrs 等方法接收数据
    app.component('counter', {
        props: {
            content: {
                type: String,
                default: "父组件传参",
            }
        },
        template: `
          <div>{{content}}</div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

#### 事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
               count: 1
            }
        },
        methods: {
            handleAdd(param ) {
                this.count += param
            }
        },
        // v-on:add="handleAdd" 监听到子组件 add 事件后执行 handleAdd，如果事件名是 v-on:add-one 需要换为驼峰 addOne
        template: `
          <div><counter v-bind:count="count" v-on:add="handleAdd"/></div>
        `
    })
    app.component('counter', {
        props: ['count'],
        // 只能触发 add 事件 并且 参数conut大于1才允许触发
        emits: {
            add: (conut) => {
                if ( conut > 1) {
                    return true
                }
                return false
            }
        },
        methods: {
            handleClick() {
                // 触发事件 10是参数
              this.$emit('add', 10)
          }
        },
        template: `
          <div v-on:click="handleClick">{{count}}</div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>

```

#### v-model

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                count: 1
            }
        },
        template: `
          <counter v-model:app="count" />
        `
    })
    app.component('counter', {
        props: ['app'],
        methods: {
            handleClick() {
                this.$emit('update:app', this.app + 3)
            }
        },
        template: `
          <div v-on:click="handleClick">{{app}}</div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

#### 修饰符

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                count: 'a'
            }
        },
        // v-model.uppercase 修饰符是 js 自带的方法 uppercase 用于转换为大写字母
        template: `
          <counter v-model.uppercase="count" />
        `
    })
    app.component('counter', {
        props: {
            'modelValue': String,
            // v-model 的修饰符
            'modelModifiers': {
                default: ()=> ({})
            }
        },
        methods: {
            handleClick() {
                let netValue = this.modelValue + 'b'
                // 判断是否使用了 uppercase
                if (this.modelModifiers.uppercase) {
                    netValue = netValue.toUpperCase()
                }
                this.$emit('update:modelValue', netValue)
            }
        },
        template: `
          <div v-on:click="handleClick">{{modelValue}}</div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

### 插槽

#### 普通插槽

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        template: `
         <myform>
           <div>提交</div>
         </myform>
         <myform>
           <button>提交</button>
         </myform>
        `
    })
    app.component('myform', {
        methods: {
          handleClick(){
              alert(123)
          }
        },
        // 传递 dom 对象的时候可以使用 slot 标签
        template: `
          <div>
            <input />
            <span v-on:click="handleClick">
              <slot></slot>
            </span>
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

#### 具名插槽

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        template: `
         <layout>
           <template v-slot:header>
             <h1>我是页头</h1>
           </template>
         </layout>
         <layout>
           <template v-slot:footer>
              <h1>我是页尾</h1>
           </template>
         </layout>
        `
    })
    app.component('layout', {
        template: `
          <div>
            <slot name="header"></slot>
            <slot name="footer"></slot>
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

#### 作用域插槽

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        // 父组件传递slotProps 子组件给 slotProps 赋值
        template: `
         <list v-slot="slotProps">
           <div>{{slotProps.item}}</div>
         </list>
        `
    })
    app.component('list', {
        data() {return {list: [1, 2, 3]}},
        template: `
          <div>
            <slot v-for="item in list" v-bind:item="item"></slot>
          </div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

### 动态组件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return { currentItem: 'input-item'}
        },
        methods: {
            handleClick() {
                if (this.currentItem === 'input-item') {
                    this.currentItem = 'common-item'
                }else {
                    this.currentItem = 'input-item'
                }
            }
        },
        // keep-alive 标签可以换成 input 内容
        template: `
          <keep-alive>
            <component v-bind:is="currentItem"></component>
          </keep-alive>
          <button v-on:click="handleClick">切换</button>
        `
    })
    app.component('input-item', {
        template: `
          <input />
        `
    })
    app.component('common-item', {
        template: `
          <div>Hello World</div>
        `
    })
    const vm = app.mount('#root')

</script>
</html>
```

### 异步组件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({

        template: `
            <div>
               <common-item />
               <a-sync-common-item />
            </div>
        `
    })
    app.component('common-item', {
        template: `
          <div>Hello World</div>
        `
    })
    app.component('a-sync-common-item', Vue.defineAsyncComponent(() => {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                resolve({
                    template: `<div>这是异步组件</div>`
                })
            },3000)
        })
    }))
    const vm = app.mount('#root')

</script>
</html>
```

## 动画渲染

### 动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
    <style>
        @keyframes leftToRight {
            0% {
                transform: translateX(-100px);
            }
            50% {
                transform: translateX(-50px);
            }
            0% {
                transform: translateX(-50px);
            }
        }

        .animation {
            animation: leftToRight 3s;
        }
    </style>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                animate: {
                    animation: false
                }
            }
        },
        methods: {
          handleClick() {
              this.animate.animation = !this.animate.animation
          }
        },
        template: `
          <div>
          <div :class="animate">hello world</div>
          <button @click="handleClick">切换</button>
          </div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>
```

### 过渡

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
    <style>
        .transition {
            transition: 3s background-color ease;
        }
    </style>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                styleObj: {
                    background: 'blue'
                }
            }
        },
        methods: {
            handleClick() {
                if (this.styleObj.background === 'blue') {
                    this.styleObj.background = 'green'
                }else {
                    this.styleObj.background = 'blue'
                }
            }
        },
        template: `
          <div>
          <div class="transition" v-bind:style="styleObj" >hello world</div>
          <button v-on:click="handleClick">切换</button>
          </div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>
```

### transition 标签

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
    <style>
        @keyframes shake {
            0% {
                transform: translateX(-100px);
            }
            50% {
                transform: translateX(-50px);
            }
            100% {
                transform: translateX(50px);
            }
        }
        .hello-leave-active {
            animation: shake 3s;
        }
        .hello-enter-active {
            animation: shake 3s;
        }
    </style>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                show: false
            }
        },
        methods: {
            handleClick() {
                this.show = !this.show
            }
        },
        template: `
          <div>
          <transition name="hello">
            <div v-if="show">hello world</div>
          </transition>
          <button @click="handleClick">切换</button>
          </div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>
```

### transition 外部引用

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
    <link
            rel="stylesheet"
            href="./animate.min.css"
    />
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({v
        data() {
            return {
                show: false
            }
        },
        methods: {
            handleClick() {
                this.show = !this.show
            }
        },
        // https://animate.style/
        template: `
          <div>
          <transition
              enter-active-class="animate__animated animate__backInDown"
              leave-active-class="animate__animated animate__backInDown"
          >
            <div v-if="show">hello world</div>
          </transition>
          <button @click="handleClick">切换</button>
          </div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>

```

### JS 动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
    <link
            rel="stylesheet"
            href="./animate.min.css"
    />
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
            return {
                show: false,
                animate: {
                    animation: false
                }
            }
        },
        methods: {
            handleClick() {
                this.show = !this.show
            },
            handleBeforeEnter(el) {
                el.style.color = 'red'
            },
            handleEnterActive(el, done) {
               const animation = setInterval(() => {
                    const color = el.style.color
                    if (color === 'red') {
                        el.style.color = 'green'
                    } else {
                        el.style.color = 'red'
                    }
                }, 1000)
                setTimeout(() => {
                    clearInterval(animation)
                    done()
                }, 5000)
            },
            handleEnterEnd() {
                alert("执行完毕")
            }

        },
        // https://animate.style/
        template: `
          <div>
          <transition
              v-bind:css="false"
              v-on:before-enter="handleBeforeEnter"
              v-on:enter="handleEnterActive"
              v-on:after-enter="handleEnterEnd"
          >
            <div v-if="show">1秒变一次颜色</div>
          </transition>
          <button @click="handleClick">切换</button>
          </div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>
```

## 高级用法

### mixins 混入

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const myMixin = {
        data() {
            return {
                number: 2,
                count: 1
            }
        }
    }
    const app = Vue.createApp({
        data() {
            return {number: 1}
        },
        // 混入 对象同时存在时组件内优先级更高
        mixins: [myMixin],
        methods: {
            handledClick() {
                console.log('handleClick')
            }
        },
        template: `
            <div>
            <div> {{number}} {{count}}</div>
            <button v-on:click="handledClick">增加</button>
            </div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>
```

### 自定义指令

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        template: `
            <div>
            <input v-focus />
            </div>
        `
    })
    app.directive('focus', {
        mounted(el) {
            el.focus()
        }
    })
    const vm = app.mount('#root')
</script>
</html>
```

### 传送门

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
    <style>
        .area{
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%);
            width: 200px;
            height: 300px;
            background: green;
        }
        .mask {
            position: absolute;
            left: 0;
            right: 0;
            top: 0;
            bottom: 0;
            background: black;
            opacity: 0.5;
        }
    </style>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        data() {
          return {
              show: false
          }
        },
        methods: {
            handleBtnClick(){
                this.show = !this.show
            }
        },
        // 通过 teleport 标签 把 mask 蒙层绑定到 body 标签下
        template: `
            <div class="area">
                <button v-on:click="handleBtnClick">按钮</button>
                <teleport to="body">
                   <div class="mask" v-show="show"></div>
                </teleport>
            </div>
        `
    })

    const vm = app.mount('#root')
</script>
</html>

```

### 渲染函数

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    // template -> render 函数 -> 虚拟 DOM -> 真实 DOM -> 页面
    const app = Vue.createApp({
        template: `
            <my-title v-bind:level="1" >
              hello
            </my-title>
        `
    })
    app.component('my-title', {
        props:['level'],
        render() {
            const {h} = Vue
            return h('h' + this.level,{},this.$slots.default)
        },
    })
    const vm = app.mount('#root')
</script>
</html>
```

### 插件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    // 可以把通用的功能封装起来
    const myPlugin = {
        install(app, options) {
            app.provide('name','zhangsen lee')
            console.log(app,options)
        }
    }
    const app = Vue.createApp({
        template: `
            <my-title/>
        `
    })
    app.component('my-title', {
        inject: ['name'],
        template: `
            <div>{{name}}</div>
        `
    })
    app.use(myPlugin, {name: 'zhangsen'})
    const vm = app.mount('#root')
</script>
</html>
```

## 组合式 API

### setup 函数

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        // 实例初始化之前创建，因此要避免使用 this
        // setup 函数返回的对象 可以在剩余组件内使用
        setup(props, context){
            return {
                name: 'zhangsen',
                handleClick:() => {
                    alert(123)
                }
            }
        },
        template: `
            <div v-on:click="handleClick">{{name}}</div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>
```

### ref 响应式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        // ref 可以处理 JS 基础类型的数据，进行响应式数据更新
        setup(props, context){
            const { ref } = Vue
            let name = ref('zhangsen')
            setTimeout(()=>{
                name.value = 'zhangsen2'
            },2000)
            return { name }
        },
        template: `
            <div>{{name}}</div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>

```

### reactive

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        // reactive可以处理 JS 非基础类型的数据
        setup(props, context) {
            const {reactive, toRefs} = Vue
            const nameObj =  reactive({name: 'zhangsen'})
            setTimeout(() => {
                nameObj.name = 'zhangsen2'
            }, 2000)
            // 转换成 ref 响应式变量
            const { name } =  toRefs(nameObj)
            return { name }
        },
        template: `
          <div>{{ name }}</div>
        `
    })
    const vm = app.mount('#root')
</script>
</html>
```

### 开发 ToDoList

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const listRelativeEffect = () => {
        const {reactive} = Vue
        const list = reactive([])
        const addItemToList = (item) => {
            list.push(item)
        }
        return {list, addItemToList}
    }
    const inputRelativeEffect = () => {
        const {ref} = Vue
        const inputValue = ref('请输入')
        const handleInputValueChange = (e) => {
            inputValue.value = e.target.value
        }
        return {inputValue, handleInputValueChange}
    }
    const app = Vue.createApp({
        setup(props, context) {
            const {list, addItemToList} = listRelativeEffect()
            const {inputValue, handleInputValueChange} = inputRelativeEffect()
            return {
                list, addItemToList,
                inputValue, handleInputValueChange
            }
        },
        template:
                `
                  <div>
                  <div>
                    <input :value="inputValue" @input="handleInputValueChange"/>
                    <button @click="() =>addItemToList(inputValue)">提交</button>
                  </div>
                  <ul>
                    <li v-for="(item,index) in list" :key="index">{{item}}</li>
                  </ul>
                  </div>
                `
    })
    const vm = app.mount('#root')
</script>
</html>

```

### 计算属性

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        setup(props, context) {
            const {ref, computed} = Vue
            const count = ref(0)
            const handleClick = () => {
                count.value += 1

            }
            let countAddFive = computed({
                get: () => {
                    return count.value + 5
                },
                set: () => {
                    return count.value = 10
                }
            })
            setTimeout(()=>{
                countAddFive.value = 100
            },3000)
            return {
                count,
                handleClick,
                countAddFive
            }
        },
        template:
                `
                    <div>
                    <span @click="handleClick">{{count}}</span> -- {{countAddFive}}
                    </div>
                `
    })
    const vm = app.mount('#root')
</script>
</html>
```

### 监听属性

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="./vue.js"></script>
</head>
<body>
<div id="root"></div>
</body>
<script>
    const app = Vue.createApp({
        setup(props, context) {
            const {ref, watch} = Vue
            const name = ref('zhangsen')
            watch(name,(currentValue,preValue) => {
                console.log(currentValue, preValue)
            })
            return { name }
        },
        template:
                `
                    <div>
                      <div>
                        Name: <input v-model="name">
                      </div>
                      <div>
                        Name is {{name}}
                      </div>
                    </div>
                `
    })
    const vm = app.mount('#root')
</script>
</html>
```

## vuex

```js
handleClick() {
    // 1. dispatch 方法派发一个 action 叫做 change
    this.$store.dispatch('change')
    // this.$store.commit('change') 同步操作时 可以简化
}


import { createStore } from 'vuex'

export default createStore({
  state: {
    name: 'zhangsen'
  },
  getters: {
  },
  mutations: {
    // 3. 感知提交更改数据
    change() {
      this.state.name = 'zhangsen1'
    }
  },
  actions: {
    // 2. 感知 change 这个 action 执行 store 中的 actions 的 change
    // 提交 change 的数据改变 
    change() {
      this.commit('change')
    }
  },
  modules: {
  }
})

```

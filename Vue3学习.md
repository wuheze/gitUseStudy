## 1、组合式API

### 1.1什么是组合式API

​       组合式 API (Composition API) 是一系列 API 的集合，使我们可以**使用函数而不是声明选项的方式**书写 Vue 组件。它是一个概括性的术语，涵盖了以下方面的 API：

- [响应式 API](https://cn.vuejs.org/api/reactivity-core.html)：例如 `ref()` 和 `reactive()`，使我们可以直接创建响应式状态、计算属性和侦听器。
- [生命周期钩子](https://cn.vuejs.org/api/composition-api-lifecycle.html)：例如 `onMounted()` 和 `onUnmounted()`，使我们可以在组件各个生命周期阶段添加逻辑。
- [依赖注入](https://cn.vuejs.org/api/composition-api-dependency-injection.html)：例如 `provide()` 和 `inject()`，使我们可以在使用响应式 API 时，利用 Vue 的依赖注入系统。

组合式 API 是 Vue 3 及 [Vue 2.7](https://blog.vuejs.org/posts/vue-2-7-naruto.html) 的内置功能。对于更老的 Vue 2 版本，可以使用官方维护的插件 [`@vue/composition-api`](https://github.com/vuejs/composition-api)。在 Vue 3 中，组合式 API 基本上都会配合 [`SFC`](https://cn.vuejs.org/api/sfc-script-setup.html) 语法在单文件组件中使用。

### 1.2为什么使用它

#### 1.2.1 更好的逻辑复用[#](https://cn.vuejs.org/guide/extras/composition-api-faq.html#better-logic-reuse)

组合式 API 最基本的优势是它使我们能够通过[组合函数](https://cn.vuejs.org/guide/reusability/composables.html)来实现更加简洁高效的逻辑复用。在选项式 API 中我们主要的逻辑复用机制是 mixins，而组合式 API 解决了 [mixins 的所有缺陷](https://cn.vuejs.org/guide/reusability/composables.html#vs-mixins)。

组合式 API 提供的逻辑复用能力孵化了一些非常棒的社区项目，比如 [VueUse](https://vueuse.org/)，一个不断成长的工具型组合式函数集合。组合式 API 还为其他第三方状态管理库与 Vue 的响应式系统之间的集成提供了一套简洁清晰的机制，例如 [RxJS](https://vueuse.org/rxjs/readme.html#vueuse-rxjs)。

#### 1.2.2更灵活的代码组织[#](https://cn.vuejs.org/guide/extras/composition-api-faq.html#more-flexible-code-organization)

许多用户喜欢选项式 API 的原因是因为它在默认情况下就能够让人写出有组织的代码：大部分代码都自然地被放进了对应的选项里。然而，选项式 API 在单个组件的逻辑复杂到一定程度时，会面临一些无法忽视的限制。这些限制主要体现在需要处理多个**逻辑关注点**的组件中，这是我们在许多 Vue 2 的实际案例中所观察到的。

我们以 Vue CLI GUI 中的文件浏览器组件为例：这个组件承担了以下几个逻辑关注点：

- 追踪当前文件夹的状态，展示其内容
- 处理文件夹的相关操作 (打开、关闭和刷新)
- 支持创建新文件夹
- 可以切换到只展示收藏的文件夹
- 可以开启对隐藏文件夹的展示
- 处理当前工作目录中的变更

这个组件[最原始的版本](https://github.com/vuejs/vue-cli/blob/a09407dd5b9f18ace7501ddb603b95e31d6d93c0/packages/@vue/cli-ui/src/components/folder/FolderExplorer.vue#L198-L404)是由选项式 API 写成的。如果我们为相同的逻辑关注点标上一种颜色，那将会是这样：

![folder component before](https://user-images.githubusercontent.com/499550/62783021-7ce24400-ba89-11e9-9dd3-36f4f6b1fae2.png)

你可以看到，处理相同逻辑关注点的代码被强制拆分在了不同的选项中，位于文件的不同部分。在一个几百行的大组件中，要读懂代码中的一个逻辑关注点，需要在文件中反复上下滚动，这并不理想。另外，如果我们想要将一个逻辑关注点抽取重构到一个可复用的工具函数中，需要从文件的多个不同部分找到所需的正确片段。

而如果[用组合式 API 重构](https://gist.github.com/yyx990803/8854f8f6a97631576c14b63c8acd8f2e)这个组件，将会变成下面右边这样：

![重构后的文件夹组件](https://user-images.githubusercontent.com/499550/62783026-810e6180-ba89-11e9-8774-e7771c8095d6.png)

现在与同一个逻辑关注点相关的代码被归为了一组：我们无需再为了一个逻辑关注点在不同的选项块间来回滚动切换。此外，我们现在可以很轻松地将这一组代码移动到一个外部文件中，不再需要为了抽象而重新组织代码，大大降低了重构成本，这在长期维护的大型项目中非常关键。

#### 1.2.3 更好的类型推导[#](https://cn.vuejs.org/guide/extras/composition-api-faq.html#better-type-inference)

近几年来，越来越多的开发者开始使用 **[TypeScript](https://www.typescriptlang.org/)** 书写更健壮可靠的代码，TypeScript 还提供了非常好的 IDE 开发支持。然而选项式 API 是在 2013 年被设计出来的，那时并没有把类型推导考虑进去，因此我们不得不做了一些[复杂到夸张的类型体操](https://github.com/vuejs/core/blob/44b95276f5c086e1d88fa3c686a5f39eb5bb7821/packages/runtime-core/src/componentPublicInstance.ts#L132-L165)才实现了对选项式 API 的类型推导。但尽管做了这么多的努力，选项式 API 的类型推导在处理 mixins 和依赖注入类型时依然不甚理想。

因此，很多想要搭配 TS 使用 Vue 的开发者采用了由 `vue-class-component` 提供的 Class API。然而，基于 Class 的 API 非常依赖 ES 装饰器，在 2019 年我们开始开发 Vue 3 时，它仍是一个仅处于 stage 2 的语言功能。我们认为基于一个不稳定的语言提案去设计框架的核心 API 风险实在太大了，因此没有继续向 Class API 的方向发展。在那之后装饰器提案果然又发生了很大的变动，在 2022 年才终于到达 stage 3。另一个问题是，基于 Class 的 API 和选项式 API 在逻辑复用和代码组织方面存在相同的限制。

相比之下，组合式 API 主要利用基本的变量和函数，它们本身就是类型友好的。用组合式 API 重写的代码可以享受到完整的类型推导，不需要书写太多类型标注。大多数时候，用 TypeScript 书写的组合式 API 代码和用 JavaScript 写都差不太多！这也让许多纯 JavaScript 用户也能从 IDE 中享受到部分类型推导功能。

#### 1.2.4 更小的生产包体积[#](https://cn.vuejs.org/guide/extras/composition-api-faq.html#smaller-production-bundle-and-less-overhead)

搭配 `<script setup>` 使用组合式 API 比等价情况下的选项式 API 更高效，对代码压缩也更友好。这是由于 `<script setup>` 形式书写的组件模板被编译为了一个内联函数，和 `<script setup>` 中的代码位于同一作用域。不像选项式 API 需要依赖 `this` 上下文对象访问属性，被编译的模板可以直接访问 `<script setup>` 中定义的变量，无需一个代码实例从中代理。这对代码压缩更友好，因为本地变量的名字可以被压缩，但对象的属性名则不能。



#### 1.2.5 对比：options API 和 composition API



**options的API**是按照**配置项的思路**进行分块的。那么，如果完成一个业务，需要使用到计算属性，watch，method等。这就导致了，一个业务的代码要阅读完，就必须在不同的配置项中进行来回翻滚。

**composition的API**是按照**业务的思路**进行分块的。这样，就会把同一个业务功能的所有代码放在了一起，方便程序员阅读修改代码。



### 1.3 第一个组合式API的例子



```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>体验组合式API</title>
</head>
<body>
  <div id="app">
    <button @click="increment">点击了{{ count }}次</button>
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, onMounted, onUpdated } = Vue

  const app = createApp({
    setup () { // 组合式API
	  //需求：在标题栏上显示组件渲染的次数。（处理count的所有代码存放在一起)
        
     //1)、定义数据count，让count成为响应式的。
      const count = ref(10)
      
      //2)、count的运算
      const increment = () => {
        console.log(1)
        count.value += 1
      }
	  //3）、初次渲染完毕时，把count的值放在页面的title上。
      onMounted(() => { // 生命周期钩子函数
        // console.log(count)
        // console.log(`初始点击次数为${count.value}`)
        document.title = `点击次数为${count.value}`
      })
        
	  //4）、当页面重新渲染时，再次把count的值放在页面的title上。
      onUpdated(() => {
        document.title = `点击次数为${count.value}`
      })

      return {
        count,
        increment
      }

    }
  })

  app.mount('#app')
</script>
</html>
```

> 体验提取公共部分
>
> ```html
> <!DOCTYPE html>
> <html lang="en">
> <head>
> <meta charset="UTF-8">
> <meta http-equiv="X-UA-Compatible" content="IE=edge">
> <meta name="viewport" content="width=device-width, initial-scale=1.0">
> <title>提取复用代码</title>
> </head>
> <body>
> <div id="app">
>  <button @click="increment">点击了{{ count }}次</button>
> </div>
> </body>
> <script src="lib/vue.global.js"></script>
> <script>
> const { createApp, ref, onMounted, onUpdated } = Vue
> 
> const useCount = () => {
>      const count = ref(10)
> 
>      const increment = () => {
>            console.log(1)
>            count.value += 1
>      }
> 
>      return {
>            count,
>            increment
>      }
> }
> 
> const useTitle = (count) => {
>      onMounted(() => { // 生命周期钩子函数
>            // console.log(count)
>            // console.log(`初始点击次数为${count.value}`)
>            document.title = `点击次数为${count.value}`
>      })
> 
>      onUpdated(() => {
>            document.title = `点击次数为${count.value}`
>      })
> }
> 
> const app = createApp({
>      setup () { // 组合式API
>            const { count, increment } = useCount() // 自定义hooks
>    
>            useTitle(count) // 自定义hooks
> 
>            return {
>              count,
>              increment
>            }
> 
>      }
> })
> 
> app.mount('#app')
> </script>
> </html>
> ```
>
> 



## 2、setup()函数

`setup()` 钩子是在组件中使用组合式 API 的入口，通常只在以下情况下使用：

1. 需要在非单文件组件中使用组合式 API 时。
2. 需要在基于选项式 API 的组件中集成基于组合式 API 的代码时。

**其他情况下，都应优先使用 [`<script setup>`](https://cn.vuejs.org/api/sfc-script-setup.html) 语法。**

### 2.1 基本使用（及其setup函数的返回值）

我们可以使用[响应式 API](https://cn.vuejs.org/api/reactivity-core.html) 来声明响应式的状态，在 `setup()` 函数中返回的对象会暴露给模板和组件实例。其它的选项也可以通过组件实例来获取 `setup()` 暴露的属性

>一、setup函数里的this不是vue对象。
>
>二、setup函数的返回值，
>
>​      1、会挂载在vue对象上
>
>​      2、可以在组件的任何地方使用（如：模板，其它配置项）

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
    <div id="app">
        <h1>msg的相关代码</h1>
        <p>msg:{{msg}}</p>
        <input type="button" value="修改msg" @click="changeMsg">
        <hr/>
        <input type="button" value="调用setup函数里的函数" @click="fn01">
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

    const {createApp,ref,watch} = Vue;

    let app = createApp({

        setup(){
            //一、 setup函数内部的this并不是vue组件。
            console.log("setup函数里的this",this);

            // 1）、定义msg
            let msg = ref("hello");
            // 2）、修改msg
            function changeMsg(){
                msg.value += 1;
            }

            //二、 setup函数的返回值，
            // 1、会挂载在vue对象上
            // 2、可以在组件的任何地方使用（如：模板，其它配置项）

            return {
                msg,
                changeMsg
            }
          
        },
        methods:{
            fn01(){
                console.log(this.msg);
                this.changeMsg();//这个函数是setup函数的返回值。
            }
        }
    })

    app.mount("#app");

</script>
```

>
>
>setup函数的返回值：是可以使用在模板上和其它配置项里的。
>
>`setup()` 自身并不含对组件实例的访问权，即在 `setup()` 中访问 `this` 并不是vue对象。你可以在选项式 API 中访问组合式 API 暴露的值（setup函数的返回值），但反过来则不行。
>
>

请注意在模板中访问从 `setup` 返回的 [ref](https://cn.vuejs.org/api/reactivity-core.html#ref) 时，它会[自动浅层解包](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#deep-reactivity)，因此你无须再在模板中为它写 `.value`。当通过 `this` 访问时也会同样如此解包。



`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>setup的基本使用</title>
</head>
<body>
  <div id="app">
    <!-- 0 -->
    {{ count }}
  </div>
</body>
<script src="lib/vue.global.js"></script>
<script>
  const { createApp, ref} = Vue

  const app = createApp({
    setup () { // 组合式API
      // 创建了响应式变量
      const count = ref(0)
     
      // 返回值会暴露给模板和其他的选项式 API 钩子
      return {
        count
      }
    },
    data () { // 虽然设置了同名的变量，但是显示的是 组合式API中的数据
      return {
        count: 100
      }
    },
    mounted () {
      console.log(this.count) // 0
    }
  })

  app.mount('#app')
</script>
</html>
```



### 2.2 setup函数的第一个参数：props

`setup` 函数的**第一个参数是组件的 `props`**。和标准的组件一致，一个 `setup` 函数的 `props` 是响应式的，并且会在传入新的 props 时同步更新。

```js
{
  props: {
    title: String,
    count: Number
  },
  setup(props) {
    console.log(props.title)
    console.log(props.count)
  }
}
```

> 请注意如果你解构了 `props` 对象，解构出的变量将会丢失响应性。因此我们推荐通过 `props.xxx` 的形式来使用其中的 props。

如果你确实需要解构 `props` 对象，或者需要将某个 prop 传到一个外部函数中并保持响应性，那么你可以使用 [toRefs()](https://cn.vuejs.org/api/reactivity-utilities.html#torefs) 和 [toRef()](https://cn.vuejs.org/api/reactivity-utilities.html#toref) 这两个工具函数：

```js
{
  setup(props) {
    // 将 `props` 转为一个其中全是 ref 的对象，然后解构
    const { title } = toRefs(props)
    // `title` 是一个追踪着 `props.title` 的 ref
    console.log(title.value)

    // 或者，将 `props` 的单个属性转为一个 ref
    const title = toRef(props, 'title')
  }
}
```

`示例`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>setup访问props</title>
</head>
<body>
  <div id="app">
    {{ count }} <button @click="count++">加1</button>
    <my-com :count="count"></my-com>
  </div>
</body>
<template id="com">
  <div>
    <h1>子组件</h1>
    {{ count }} - {{ doubleCount }}
  </div>
</template>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, computed, toRefs, toRef } = Vue

  const Com = {
    template: '#com',
    props: {
      count: Number
    },
    setup (props) {
      console.log(props)
      // 1.计算属性 - 响应式
      // const doubleCount = computed(() => props.count * 2)

      // 2.不要把props中的数据解构，解构会丢失响应式
      // const { count } = props // 模板中 doubleCount 始终为 0
      // const doubleCount = computed(() => count * 2)

      // 3.如果既需要解构，还要保持响应式 toRefs
      // const { count } = toRefs(props) // 将 `props` 转为一个其中全是 ref 的对象，然后解构
      // // console.log(count)
      // const doubleCount = computed(() => count.value * 2) // ref对象访问值通过 value 属性

      // 4.保持响应式 toRef
      const count = toRef(props, 'count') // 将 `props` 的单个属性转为一个 ref
      // console.log(count)
      const doubleCount = computed(() => count.value * 2)

      return {
        doubleCount
      }
    }
  }

  const app = createApp({
   components: {
    MyCom: Com
   },
   setup () {
    const count = ref(0)

    return {
      count
    }
   }
  })

  app.mount('#app')
</script>
</html>
```

### 2.3 setup函数的第二个参数：上下文对象

传入 `setup` 函数的第二个参数是一个 **Setup 上下文**对象。上下文对象暴露了其他一些在 `setup` 中可能会用到的值：

```
{
  setup(props, context) {
  	
    // 透传的 Attributes（非响应式的对象，等价于 $attrs（拿到透传的属性））
    console.log(context.attrs)

    // 插槽（非响应式的对象，等价于 $slots）
    console.log(context.slots)

    // 触发事件（函数，等价于 $emit）
    console.log(context.emit)

    // 暴露公共属性（函数）
    console.log(context.expose)
    
  }
}
```

该上下文对象是非响应式的，可以安全地解构：

```
{
  setup(props, { attrs, slots, emit, expose }) {
    ...
  }
}
```

`attrs` 和 `slots` 都是有状态的对象，它们总是会随着组件自身的更新而更新。这意味着你应当避免解构它们，并始终通过 `attrs.x` 或 `slots.x` 的形式使用其中的属性。此外还需注意，和 `props` 不同，`attrs` 和 `slots` 的属性都**不是**响应式的。如果你想要基于 `attrs` 或 `slots` 的改变来执行副作用，那么你应该在 `onBeforeUpdate` 生命周期钩子中编写相关逻辑。

`expose` 函数用于显式地限制该组件暴露出的属性，当父组件通过[模板引用](https://cn.vuejs.org/guide/essentials/template-refs.html#ref-on-component)访问该组件的实例时，将仅能访问 `expose` 函数暴露出的内容（vue2（选项式api）中，使用this.$refs 可以拿到子组件的所有的属性和方法在父组件中都可以使用）

`示例：`

attrs, slots, emit

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
    <div id="app">
        <input type="button" value="修改name" @click="changeName">
        <book id="book01" class="cls"  :name="name" author="罗贯中" @change="changeFn">
            <template #default>
                <div>默认插槽的内容</div>
            </template>
            <template #footer>
                <div>footer插槽的内容</div>
            </template>
        </book>        
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>

<script>    

    const {createApp,ref,computed} = Vue;


    // 定义一个组件：
    let book = {
        template:`
            <h1>书籍信息：</h1>
            <p>书名:{{name}}</p>
            <slot>默认插槽</slot>
            <p>作者:{{author}}</p>
            <slot name="footer">footer插槽</slot>
        `,        
        props:["name","author"],
        emits:["change"],
        setup(props,context){                          
            // 1、context.attrs: 是透传过来的属性，this.$attrs
            // console.log("book组件setup函数的context参数attrs",context.attrs);      
            // console.log(context.attrs.id);          
            // console.log(context.attrs.class);    

            // 2、context.slots：是插槽，this.$slots
            // console.log("book组件setup函数的context参数slots",context.slots);
            // console.log(context.slots.default());
            // console.log(context.slots.footer());

            // 3、context.emit：是触发事件的函数，this.$emit()
            // console.log("book组件setup函数的context参数emit",context.emit);
            context.emit("change",{name:"任晨阳"});

            // 4、context.expose：是父组件通过ref得到的子组件对象可以访问的属性和方法。（在下一个demo中写）
            
        }
    }

    let app = createApp({
        setup(){
            let name = ref("三国演义");
            function changeName(){
                name.value += "1";
            }
            function changeFn(obj){
                console.log("changeFn",obj);
            }
            return {
                name,changeName,changeFn
            }
        },
        components:{
            book
        }   
    })

    

    app.mount("#app");



</script>
```

`示例：`

expose

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
    <div id="app">
        <input type="button" value="选项式api中获得子组件对象" @click="getChild">
        <!-- 2）、在模板上用ref属性，给ref属性赋值为 bookref -->
        <book ref="bookref"> </book>        
        <input type="button" value="组合式api中获得子组件对象" @click="getChildComposition">
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>

<script>    

    const {createApp,ref,computed} = Vue;


    // 定义一个组件：
    let book = {
        template:`
            <h1>书籍信息：</h1>
            <p>书名:{{name}}</p>
            <slot>默认插槽</slot>
            <p>作者:{{author}}</p>
            <slot name="footer">footer插槽</slot>
        `,
        setup(props,context){       
            let msg = ref("hello");                               
            let str = ref("嘿嘿");

            function sonFn(args){
                console.log("sonFn.args",args);
            }
            
            // 4、context.expose：是父组件通过ref得到的子组件对象可以访问的属性和方法
            context.expose({
                str,sonFn
            })

            // return的内容，会挂载在vue对象上。但是，这个并不会开放给父组件
            return {
                msg
            }
        },
        // data(){
        //     return {
        //         bookmsg:"hello，我是book"
        //     }
        // },
        // methods:{
        //     bookFn(){
        //         console.log("bookFn");
        //     }
        // }
    }

    let app = createApp({
        setup(){
            // 在组合式api中使用ref，获取子组件实例
            // 1）、定义一个ref对象（ref函数的返回值）；
            let bookref = ref(null);

            // 2）、在模板上用ref属性，给ref属性赋值为 bookref
            //  这个注释在模板上找代码。


            function getChildComposition(){
                console.log("bookref",bookref);  
                console.log("bookref.value.str",bookref.value.str)//可以访问子组件的str，因为，子组件用 context.expose对外开放了
                console.log("bookref.value.msg",bookref.value.msg)//不能使用，因为没有开放

                //可以访问子组件的sonFn，因为，子组件用 context.expose对外开放了
                bookref.value.sonFn({
                    name:"王义鑫",
                    sex:"男"
                });
            }

            return {
                bookref,getChildComposition
            }
           
        },
        components:{
            book
        },
        // methods:{
        //     getChild(){
        //         // this.$refs.bookRef:可以获取到子组件的实例。
        //         const son = this.$refs.bookref;
        //         console.log("son.bookmsg",son.bookmsg);
        //         son.bookFn();
        //     }
        // }
    })

    app.mount("#app");

</script>
```



> 在父组件通过ref获取子组件的实例的属性和方法的需求中，需要注意：
>
> 1. 如果子组件是 选项式API组件，基本不需要做任何操作
>
> 2. 如果子组件是 组合式API组件，需要通过 context.expose 暴露给父组件需要使用的属性和方法
>
> 3. 如果父组件使用 选项式API, 可以通过 this.$refs.refName 访问到子组件想要你看到的属性和方法
>
> 4. 如果父组件使用 组合式API,需要在setup中先创建 refName，然后再访问子组件想要你看到的属性和方法（const refName = ref()    refName.value.X）



解释：

>setup函数里的：context.expose和return。
>
>1、 context.expose：对本组件之外的开放，父组件可以使用这些属性和方法
>
>2、return：setup的返回值，本组件内部开放，开放给本组件的其它配置项和模板上。
>
>​                     可以在本组件里使用（本组件的模板上，和本组件的其它配置项里）
>
>



### 2.4 与渲染函数一起使用

`setup` 也可以返回一个[渲染函数](https://cn.vuejs.org/guide/extras/render-function.html)，此时在渲染函数中可以直接使用在同一作用域下声明的响应式状态：

```
{
  setup() {
    const count = ref(0)
    return () => h('div', count.value)
  }
}
```

返回一个渲染函数将会阻止我们返回其他东西。对于组件内部来说，这样没有问题，但如果我们想通过模板引用将这个组件的方法暴露给父组件，那就有问题

我们可以通过调用 [`expose()`](https://cn.vuejs.org/api/composition-api-setup.html#exposing-public-properties) 解决这个问题：

```
{
  setup(props, { expose }) {
    const count = ref(0)
    const increment = () => ++count.value

    expose({
      increment
    })

    return () => h('div', count.value)
  }
}
```

`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>setup与渲染函数一起</title>
  <style>
    .box{
      width: 100px;
      height: 100px;
      border:2px solid red;
    }
  </style>
</head>
<body>
  <div id="app">
    <button @click="add">加1</button>
    <my-com ref = "com"></my-com>
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>

  const { createApp, ref, h } = Vue;

  const Com = {
    setup (props, { expose }) {
      const count = ref(0)

      const increment = () => {
        count.value += 1
      } 

      expose({ // 关键
        increment
      })

      return () => h('div', { class: 'box' }, count.value)
    }
  }

  const app = createApp({
    components: {
      MyCom: Com
    },
    setup () { // 组合式API

      // 获取组件对象
      const com = ref()

      const add = () => {
        com.value.increment()
      }
      return {
        com,
        add
      }

    }
  })

  app.mount('#app')
</script>
</html>
```



## 3、组合式api的相关函数

### 1、响应式核心

#### 1）、 ref(值)

​     **1）、功能：**接受值，返回一个响应式的、可更改的 ref 对象，ref对象只有一个属性：`value`。value属性保存着接受的值。

​     **2）、使用ref对象：**模板上不需要写 .value 属性（会自动解构），在js中，使用 .value 来完成**数据的读写**。

​     **3）、ref接收基本类型和引用类型**

- ref可以接收基本类型。
- ref也可以接收引用类型：如果将一个对象传给 ref函数，那么这个对象将通过 [reactive()](https://cn.vuejs.org/api/reactivity-core.html#reactive) 转为具有深层次响应式的对象。    



```
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

`示例`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">
        <!-- ref响应对象，在模板上使用时，不需要.value，因为，在模板上vue会自动处理 -->
        <p>msg：{{msg}}</p>
        <input type="button" value="修改msg" @click="changeMsg">
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    
    
   // ref：对应着选项式api中data（data中的数据都是响应式的）
    

    const {createApp,ref} = Vue;

    let app = createApp({
        setup(){
            // ref()函数会返回一个响应式对象，该响应式对象的value属性，保存着数据的值
            // 1、在模板使用时，不需要 .value
            // 2、在js中使用时，需要 .value
            let msg=ref("hello");//ref函数返回一个响应式的对象，该对象的value属性的值是"hello"
            console.log("msg",msg);
            console.log("msg.value",msg.value);

            function changeMsg(){
                msg.value+="1";
            }

            return {
                msg,changeMsg
            }
        }
    })

    app.mount("#app");

</script>
```

> 以后创建 非 对象类型的数据 使用  ref， 创建对象类型的数据建议使用 reactive

#### 2）、 reactive()

​      1）、**功能：** 接受一个对象，返回一个对象的响应式代理（proxy）。返回的对象以及其中嵌套的对象都会通过 [ES Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 包裹，因此**不等于**源对象，**建议只使用响应式代理，避免使用原始对象。**

​          **响应式转换是“深层”的：**它会影响到所有嵌套的属性。一个响应式对象也将深层地解包任何 [ref](https://cn.vuejs.org/api/reactivity-core.html#ref) 属性，同时保持响应性。

​     2）、**注意点：**当访问到某个响应式数组或 `Map` 这样的原生集合类型中的 ref 元素时，不会执行 ref 的解包。



创建一个响应式对象：

```
const obj = reactive({ count: 0 })
obj.count++
```

##### 2.1）、reactive的基本使用，对象使用reactive

`reactive示例：`

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">
        <!-- // 1、引用类型的数据使用reactive       -->
        <p>person.name:{{person.name}}</p>
        <p>person.sex:{{person.sex}}</p>
        <p>person.age:{{person.age}}</p>
        <input type="button" value="修改age" @click="changeAge">
        <p>person.wife.age:{{person.wife.age}}</p>
        <input type="button" value="修改wife.age" @click="changeWifeAge">
        <hr/>
        <!--  // 2、引用类型的数据使用ref -->
        <p>book.name:{{book.name}}</p>
        <p>book.price:{{book.price}}</p>
        <input type="button" value="修改价格" @click="changePrice">
        <p>book.author.age:{{book.author.age}}</p>
        <input type="button" value="修改作者的age" @click="changeAuthorAge">
        <hr/>
        <!--   // 3、基本类型使用reactive：不会成为响应式 -->
        <p>msg:{{msg}}</p>
        <input type="button" value="修改Msg" @click="changeMsg">
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

   // reactive：对应着选项式api中data，一般使用在对象上
   // ref也可以让对象成为响应式的，只不过需要多些一个value。


    const {createApp,ref,reactive} = Vue;

    let app = createApp({
        setup(){

            // reactive()函数会返回代理对象Proxy。
            // 1、引用类型的数据使用reactive
            let person = reactive({
                name:"王义鑫",
                sex:"男",
                age:18,
                wife:{
                    name:"范冰冰",
                    age:16
                }
            });

            console.log("person",person); //Proxy对象

            function changeAge(){
                person.age++;
            }
            function changeWifeAge(){
                person.wife.age++;
            }
           
            // 2、引用类型的数据使用ref
            let book = ref({
                name:"三国演义",
                price:51.2,
                author:{
                    name:"罗贯中",
                    age:108,
                    address:"南窑头国际"
                }
            });

            function changePrice(){
                book.value.price ++;
            }
            function changeAuthorAge(){
                book.value.author.age++;
            }

            // 3、基本类型使用reactive：不会成为响应式
            let msg = reactive("hello");//如果给reactive传入基本类型的数据，那么，返回值就是基本类型
            console.log("msg",typeof msg);//string。肯不是响应式的了

            function changeMsg(){
                msg +="1";
                console.log("changeMsg函数",msg);
            }

            return {
                person,changeAge,changeWifeAge,
                book,changePrice,changeAuthorAge,
                msg,changeMsg
            }
        }
    })

    app.mount("#app");

</script>
```



##### 2.2）把ref对象赋给reactive，会自动解包



```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">      
        <p>countRef:{{countRef}}</p>
        <p>personReactive.count:{{personReactive.count}}</p>
        <input type="button" value="修改count" @click="changeCount">
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

    // 把ref对象赋给reactive，会自动解包。
    // 进一步解释：把ref对象作为某个对象的属性值，并把该对象传给reactive，那么ref会自动解包

    const {createApp,ref,reactive} = Vue;

    let app = createApp({
        setup(){

            // 1、定义一个ref对象
            let countRef = ref(12);
            

            // 2、把ref对象作为某个对象的属性值，并把该对象传给reactive，
            // 1）、那么ref会自动解包（把value属性取出来）
            // 2）、ref和对象的属性形成了关联关系。
            let personReactive = reactive({
                count:countRef //会解包，并且还会把count属性和countRef对象关联起来。
            });            
                        
            console.log("personReactive.count",personReactive.count);//12；其实是代理对象读取count属性时，调用了get，get函数里返回的是ref对象的value属性
            console.log(personReactive.count===countRef);//false
            console.log(personReactive.count===countRef.value);//true    

            function changeCount(){
                personReactive.count++;//修改reactive对象的属性时，ref的值也会变化。
                console.log("countRef.value",countRef.value);
            }

            // 3、如果没有reactive，肯定不会解包
            let obj = {
                count:countRef
            }

            console.log("obj.count",obj.count);//ref对象           

            return {
                countRef,personReactive,changeCount
            }
        }
    })

    app.mount("#app");

</script>
```



##### 2.3）、注意：

当访问到某个响应式数组或 `Map` 这样的原生集合类型中的 ref 元素时，**不会**执行 ref 的解包：

```
const books = reactive([ref('Vue 3 Guide'),ref(5)])//给reactive传入的是数组。不会解包
// 这里需要 .value
console.log(books[0].value)//'Vue 3 Guide'
console.log(books[1].value)//5

const map = reactive(new Map([['count', ref(0)]]))//给reactive传入的是Map。不会解包
// 这里需要 .value
console.log(map.get('count').value)
```



`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">      
        <ul>
            <li v-for="item in booksReactive">
                <!-- 
                    如果reactive对象里写的是ref数组时。
                    显示ref对象的值时，需要使用 .value
                -->
                <p>{{item.value}}</p>
            </li>
        </ul>
        <input type="button" value="添加一本书" @click="addBook">
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

//   reactive接收ref的数组时，不会解包

    const {createApp,ref,reactive} = Vue;

    let app = createApp({
        setup(){

            let bookRef1 = ref("三国演义");
            let bookRef2 = ref("红楼梦");

            let booksReactive = reactive([bookRef1,bookRef2]);
            
            console.log("booksReactive",booksReactive);

            console.log("booksReactive[0]",booksReactive[0]);//ref对象
            console.log("booksReactive[1]",booksReactive[1]);//ref对象

            console.log("booksReactive[0].value",booksReactive[0].value);

            function addBook(){
                booksReactive.push(ref("西游记"));
            }

            return {
                booksReactive,addBook
            }
        }
    })

    app.mount("#app");

</script>
```

> 
>
> reactive  和 ref的选用：
>
> 
>
> 对象用reactive，其它用ref
>
> 



#### 3）、 readonly()

1）、**功能：**接受一个对象 (不论是响应式还是普通的) 或是一个 [ref](https://cn.vuejs.org/api/reactivity-core.html#ref)，返回一个原值的只读代理。

2）、**只读代理是深层的：**对任何嵌套属性的访问都将是只读的。它的 ref 解包行为与 `reactive()` 相同，但解包得到的值是只读的。

```
const original = reactive({ count: 0 })

const copy = readonly(original)

watchEffect(() => {
  // 用来做响应性追踪
  console.log(copy.count)
})

// 更改源属性会触发其依赖的侦听器
original.count++

// 更改该只读副本将会失败，并会得到一个警告
copy.count++ // warning
```



`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">      
        <p>bookReadonly.name:{{bookReadonly.name}}</p>
        <p>bookReadonly.author.name:{{bookReadonly.author.name}}</p>
        <input type="button" value="修改" @click="changeName" >
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

    // readonly 接收一个对象（响应式对象，普通对象）和ref。让其成为只读的，而且，只读是深层次的。

    // 原生中学习的const的只读，只限定变量对应的内存区域是只读的。
    const p = {
        name:"王义鑫"
    }

    p = {name:"罗怡欣"};//不能改的
    console.log("p.name",p.name);//王义鑫

    p.name = "张伟业";//可以
    console.log("p.name",p.name);//张伟业


    const {createApp,ref,reactive,readonly} = Vue;

    let app = createApp({
        setup(){

            let book = {
                name:"三国演义",
                author:{
                    name:"老罗",
                    address:"南窑头国际"
                }
            }

            let bookReactive = reactive(book);

            let bookRef = ref(book);

            // let bookReadonly = readonly(book);
            // let bookReadonly = readonly(bookReactive);
            let bookReadonly = readonly(bookRef);

            console.log("bookReadonly",bookReadonly);

            function changeName(){
                // bookReadonly.name +=1;
                // console.log("bookReadonly.name",bookReadonly.name);
                // bookReadonly.author.name += 1;
                // console.log("bookReadonly.author.name",bookReadonly.author.name);

                bookReadonly.value.name +=1;
                console.log("bookReadonly.value.name",bookReadonly.value.name);
                
            }
           
            return {
                bookReadonly,changeName
            }
        }
    })

    app.mount("#app");

</script>
```

#### 4）、 computed () 

- **功能：**computed是计算属性。和选项式api中的计算属性实现的功能一样。

- **参数：**

  - 可以接受一个 getter 函数，返回一个只读的响应式 [ref](https://cn.vuejs.org/api/reactivity-core.html#ref) 对象。该 ref 通过 `.value` 暴露 getter 函数的返回值。

  - 也可以接受一个带有 `get` 和 `set` 函数的对象来创建一个可写的 ref 对象。

  

  

##### 4.1）、创建一个只读的计算属性 ref：

```
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // 错误
```

##### 4.2）、创建一个可写的计算属性 ref：

```
const count = ref(1)
const plusOne = computed({
  get: () => count.value + 1,
  set: (val) => {
    count.value = val - 1
  }
})

plusOne.value = 1
console.log(count.value) // 0
```

`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">      
        <!-- 1、定义一个只读的计算属性 -->
        <p>count:{{count}}</p>
        <p>doubleCount：{{doubleCount}}</p>
        <input type="button" value="修改count" @click="changeCount" >
        <hr/>
        <!--  2、定义一个可读写的计算属性 -->

        <p>age:{{age}}</p>
        <p>wifeAge:{{wifeAge}}</p>
        <input type="button" value="修改age" @click="changeAge" >
        <input type="button" value="修改wifeAge" @click="changeWifeAge" >       

    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

     const {createApp,ref,reactive,computed} = Vue;

    let app = createApp({
        setup(){

            // 1、定义一个只读的计算属性
            let count = ref(5);

            // 定义一个计算属性：
            let doubleCount = computed(()=>count.value*count.value);

            function changeCount(){
                count.value ++;
            }

            // 2、定义一个可读写的计算属性

            let age = ref(20);

            let wifeAge = computed({
                set:function(newVal){
                    console.log("newVal",newVal);
                    age.value = newVal-3;
                },
                get:function(){
                   return age.value+3 
                }
            })

            function changeAge(){
                age.value--;
            }
            
            function changeWifeAge(){
                wifeAge.value--;
            }

            return {
                count,doubleCount,changeCount,
                age,wifeAge,changeAge,changeWifeAge
            }
        }
    })

    app.mount("#app");

</script>
```



#### 5）、 watch()

```js
watch(第一个参数，第二个参数，第三个参数)
```



**功能：**侦听数据的变化，和选项式api中的watch实现的功能一样，组合式api中watch功能更加强大，灵活。**默认是懒侦听的**，即仅在侦听源发生变化时才执行回调函数。

**参数：**

- **第一个参数**：侦听器的**源**，可以是以下几种：
  - 一个函数（返回一个值的函数）
  - 一个 ref
  - 一个响应式对象
  - ...或是由以上类型的值组成的数组

- **第二个参数：**在（第一个参数的值）发生变化时要调用的**回调函数**。这个回调函数接受三个参数：新值、旧值，以及一个用于**注册副作用清理的回调函数**。该回调函数会在副作用**下一次重新执行前调用**，可以用来清除无效的副作用，例如：等待中的异步请求。

当侦听多个来源时，回调函数接受两个数组，分别对应来源数组中的新值和旧值。

- **第三个参数：**可选的， 是一个对象，支持以下这些选项：
  - **`immediate`**：在侦听器创建时立即触发回调。第一次调用时旧值是 `undefined`。

  - **`deep`**：如果源是对象，强制深度遍历，以便在深层级变更时触发回调。参考[深层侦听器](https://cn.vuejs.org/guide/essentials/watchers.html#deep-watchers)。



与 [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core.html#watcheffect) 相比，`watch()` 使我们可以：

- 懒执行副作用；
- 更加明确是应该由哪个状态触发侦听器重新执行；
- 可以访问所侦听状态的前一个值和当前值。

- **示例**

  侦听一个 ref（侦听ref 不用写value）：

  ```
  const count = ref(0)
  watch(count, (count, prevCount) => {
    /* ... */
  })
  ```

  侦听一个 getter 函数：

```
  const state = reactive({ count: 0 })
watch(
    () => state.count,
    (count, prevCount) => {
      /* ... */
    }
  )
```

  当侦听多个来源时，回调函数接受两个数组，分别对应来源数组中的新值和旧值：

  ```
  watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
    /* ... */
  })
  ```

  **当使用 getter 函数作为源时**，回调只在此函数的返回值变化时才会触发。如果你想让回调在深层级变更时也能触发，你需要使用 `{ deep: true }` 强制侦听器进入深层级模式。在深层级模式时，如果回调函数由于深层级的变更而被触发，那么新值和旧值将是同一个对象。

  ```
  const state = reactive({ count: 0 })
  watch(
    () => state,
    (newValue, oldValue) => {
      // newValue === oldValue
    },
    { deep: true }
  )
  ```

  当直接侦听一个响应式对象时，侦听器会自动启用深层模式：

  ```
  const state = reactive({ count: 0 })
  watch(state, () => {
    /* 深层级变更状态所触发的回调 */
  })
  ```

`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">      
        <!--  1、监听一个ref。 -->
        <p>age:{{age}}</p>
        <p>wifeAge:{{wifeAge}}</p>
        <input type="button" value="修改age" @click="changeAge" >
        <hr/>

        <!--  2、监听一个响应式对象 -->

        <p>person.name:{{person.name}}</p>
        <p>person.wife.name:{{person.wife.name}}</p>
        <input type="button" value="修改wife的name" @click="changeWifeName" >
        <hr/>  

        <!--  3、监听一个回调函数 -->       
        
        <p>count:{{count}}</p>         
        <input type="button" value="修改count" @click="changeCount" />

        <p>inc:{{inc}}</p> 
        <input type="button" value="修改inc" @click="changeInc" />
        <hr/>

        <!-- 4、体现deep （当返回值是一个对象时，需要使用deep:true -->
        <p>a:{{objReactive.a}}</p>
        <p>b.b1:{{objReactive.b.b1}}</p>
        <input type="button" value="修改deep" @click="changeDeep">
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

     const {createApp,ref,reactive,watch} = Vue;

    let app = createApp({
        setup(){

            // 1、监听一个ref。

            let age = ref(18);

            let wifeAge = ref(0);

            watch(age,function(newVal,oldVal){
                console.log("watch",newVal,oldVal);
                wifeAge.value = newVal + 3;
            },{
                immediate:true
            })

            function changeAge(){
                age.value--;
            }

            // 2、监听一个响应式对象

            let person = reactive({
                name:"王义鑫",
                age:12,
                wife:{
                    name:"范冰冰"                    
                }
            })
            
            watch(person,function(){
                console.log("watch person");
            })

            function changeWifeName(){                
                console.log("changeWifeName");
                
                // person = {};//这样修改，不会监听到，因为，你已经让person变成了普通对象，而不是proxy对象

                // person.wife = {};//可以
                // person.wife.name="李冰冰";//可以
            }

            // 3、监听一个回调函数
            let count = ref(12);
            let inc = ref(2);

            // 因为回调函数里使用了count和inc，所以，相当于监听了 count和inc的变化
            watch(()=>count.value+inc.value,function(newVal){
          // watch([count,inc],function(newVal){
                console.log("watch监听回调函数newVal",newVal);
            },{
                immediate:true
            });
                
            function changeCount(){
                count.value ++;
            }

            function changeInc(){
                inc.value ++;
            }

            // 4、体现deep

            let objReactive = reactive(
                {
                    a:1,
                    b:{
                        b1:2,
                        b2:3
                    }
                }
            )

            // 当监听的源是回调函数时，只有在 回调函数的返回值发生变化时，才触发watch。
            // 那么，如果：返回值是引用类型，那么地址不变，返回值就不会变。所以说,改变对象的属性时,并不会引起watch。
            // 所以说:  当回调函数（侦听源）的返回值是(响应式)对象时。需要使用deep。
            watch(()=>objReactive,function(){
                console.log("watch,deep");
            },{
                deep:true
            })

            function changeDeep(){
                console.log("changeDeep");
                // objReactive.a ++;//并没有改变 objReactive对象，只是改变了对象的属性。
                objReactive.b.b1 ++;
            }
            
            
            return {
              age,wifeAge,changeAge,
              person,changeWifeName,
              count,inc,changeCount,changeInc,
              objReactive,changeDeep
            }
        }
    })

    app.mount("#app");

</script>
```





#### 6）、 watchEffect()

[watchEffect](https://cn.vuejs.org/api/reactivity-core.html#watcheffect)

```js
function watchEffect(
  effect: (onCleanup: OnCleanup) => void,
  options?: WatchEffectOptions
): StopHandle
```



- **功能：** watchEffect也是监听数据，但是它会立即运行一个函数，而不是懒侦听。watchEffect的侦听（依赖）的数据：watchEffect里使用了哪个数据，哪个数据就是watchEffect的依赖。

- **参数：**
  - **第一个参数：**要运行的副作用函数。这个副作用函数的参数也是一个函数，**注册副作用清理的回调函数**。该回调函数会在副作用**下一次重新执行前调用**，可以用来清除无效的副作用，例如：等待中的异步请求。（和watch的第二个参数中回调函数的第三参数一样）。
  - **第二个参数：**可选的选项，可以用来调整副作用的刷新时机或调试副作用的依赖。因为，侦听默认是在vue组件更新前调用，如果你希望组件更新后调用，可以把第二个参数传入：`{  flush: 'post' }`

- **返回值：**用来停止该副作用的函数。

  

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> 输出 0

count.value++
// -> 输出 1
```



##### 6.1）、基本使用（副作用）

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">      
        <p>age:{{age}}</p>
        <p>isAdult：{{isAdult}}</p>
        <input type="button" value="修改Age" @click="changeAge" >
        
        <p>count:{{count}}</p>
        <p>isLimit:{{isLimit}}</p>
        <p>limitChina:{{limitChina}}</p>
        <input type="button" value="修改Count" @click="changeCount" >
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

     const {createApp,ref,reactive,watch,watchEffect,computed} = Vue;

    let app = createApp({
        setup(){

            let age = ref(16);
            let isAdult = ref(false);

            let count = ref(5);
            let maxCount = 10;
            let isLimit = ref(false);//是否达到上限
            let limitChina = computed(()=>isLimit.value?"不能再买了":"继续买");

            // 1、watchEffect回调函数里，使用哪个响应式数据，那么就会监听哪个数据。

            watchEffect(()=>{                
                console.log("watchEffect：age.value",age.value);
                // 此处写的是 age发生变化时的副作用（即：当age发生变化，还应该做什么事情）
                isAdult.value = age.value>=18?true:false;

                // 此处写的是：count发生变化时的副作用。
                isLimit.value = count.value>=maxCount?true:false;

            });

            // watch(age,function(){
            //     console.log("watch：age.value",age.value);
            // })

            function changeAge(){
                age.value++;
            }

            function changeCount(){
                
                if(isLimit.value){
                    return;
                }

                count.value++;
            }
            
            return {
                age,changeAge,isAdult,
                count,changeCount,isLimit,limitChina
            }
        }
    })

    app.mount("#app");

</script>



```





##### 6.2）、副作用清除：

下面的示例，有点像防抖。

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>    
    <div id="app">      
        <p>count:{{count}}</p>
        <p>isLimit：{{isLimit}}</p>
        <input type="button" value="修改" @click="changeCount" >
    </div>
</body>
</html>

<script src="./js/vue.global.js"></script>
<script>    

     const {createApp,ref,reactive,watchEffect} = Vue;

    let app = createApp({
        setup(){

            let count = ref(1);
            let isLimit = ref(false);

            watchEffect((cb)=>{
                console.log("watchEffect：count.value",count.value);

                let myTimer = setTimeout(function(){

                    console.log("修改isLimit的值");
                    
                    isLimit.value = count.value>=10?true:false

                },2000);

                // cb里的回调函数是在下次触发副作用时，首先会执行的代码。
                cb(()=>{
                    console.log("清除定时器");
                    clearTimeout(myTimer);
                })

            });

            function changeCount(){
                count.value++;
            }

            
            return {
               count,changeCount,isLimit
            }
        }
    })

    app.mount("#app");

</script>
```



##### 6.3）、停止侦听器：

```js
 const stop = watchEffect(() => {
     console.log(count.value)
 })
 // 当不再需要此侦听器时:
 const stopWatch = () => {
     stop()
 }
```

`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>watchEffect</title>
</head>
<body>
  <div id="app">
    <button @click="increment">点击了{{ count }}次</button>
    <p id="p01">p01的内容：{{msg}}</p><button @click="addMsg">点击了addMsg</button>
    <button @click="stopWatch">停止监听</button>

    <ul>
      <li @click="id=1">请求第一条数据</li>
      <li @click="id=2">请求第二条数据</li>
      <li @click="id=3">请求第三条数据</li>
    </ul>
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, watchEffect } = Vue

  const app = createApp({

    setup () {
      const count = ref(0)

      const increment = () => {
        count.value += 1
      }

      const msg = ref("a");

      const addMsg = ()=>{
        msg.value += 1;
      }

      // 返回值可以用来停止侦听
      const stop = watchEffect(() => {
        console.log(document.getElementById("p01").innerHTML);//此处打印更新前还是更新后的值，由第二个参数决定
        console.log(`监听到count的数据为${count.value}`)
        console.log(`监听到msg的数据为${msg.value}`)
      },{
        flush:"post"
      })

      const stopWatch = () => {
        stop()
      }

      const id = ref(1)

      watchEffect((onCleanup) => {
        console.log("id.value",id.value) // 关键  ---  引起 当前 watchEffect 二次执行
        const timer = setTimeout(() => {
          console.log(`请求第${id.value}条数据`)
        }, 3000)
        console.log("timer",timer);

        // onCleanup的回调函数 会在id更改优先调用。如：下次id更改时，可以对上次的未完成的事情（如：请求）做清除……
        onCleanup(() => {
          console.log('onCleanup的回调函数被调用了','清除')
          clearTimeout(timer);//在id的值更新比较频繁时，只让最后一次请求起作用  
        })

      })

      // onInvalidate
      return {
        count,
        increment,
        msg,
        addMsg,
        stopWatch,
        id
      }
    }
  })

  app.mount('#app')
</script>
</html>
```

> watchEffect没有具体监听哪一个值的变化，只要内部有某一个状态发生改变就会执行





### 2、工具函数

#### 1）、 toRef() 

[官网](https://cn.vuejs.org/api/reactivity-utilities.html#toref)

​       基于响应式对象上的一个属性，创建一个对应的 ref。这样创建的 ref 与其源属性保持同步：改变源属性的值将更新 ref 的值，反之亦然。



`示例：`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>toRef</title>
</head>
<body>
  <div id="app">
    
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, toRef, reactive } = Vue

  const app = createApp({
    setup () {
      // 基于响应式对象上的一个属性，创建一个对应的 ref。
      // 这样创建的 ref 与其源属性保持同步：改变源属性的值将更新 ref 的值，反之亦然。
      const state = reactive({
        foo: 1,
        bar: 2
      })

      //创建一个ref，让foo和state.foo的值保存一致。
      const count = toRef(state, 'foo');//如果用ref，那么，不会同步

      console.log(count.value) // 1，这就是state.foo的值。

      count.value++
      console.log("count.value++后，state.foo",state.foo) // 2
      console.log("count.value++后，count.value",count.value) // 2

      state.foo++
      console.log("state.foo++后，state.foo：",state.foo) // 3
      console.log("state.foo++后，count.value：",count.value) // 3
      return {

      }

    }
  })

  app.mount('#app')
</script>
</html>
```

>如果用ref，那么，不会同步



#### 2）、 toRefs()

将一个响应式对象转换为一个普通对象，这个普通对象的每个属性都是指向源对象相应属性的 ref。每个单独的 ref 都是使用 [`toRef()`](https://cn.vuejs.org/api/reactivity-utilities.html#toref) 创建的。

`示例：`

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>toRef</title>
</head>

<body>
  <div id="app">
    <p>foo:{{foo}}</p>
    <input type="button" value="修改foo" @click="foo+='1'" >
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, toRefs, reactive } = Vue

  const app = createApp({
    setup() {
      function useFeatureX() {
        
       const state = reactive({
          foo: 1,
          bar: 2
        })
        // ...基于状态的操作逻辑

        // 在返回时都转为 ref
        return toRefs(state)
      }

      // 可以解构而不会失去响应性
      const { foo, bar } = useFeatureX()

      return {
        foo
      }

    }
  })

  app.mount('#app')
</script>

</html>
```



> `toRefs` 在调用时只会为源对象上可以枚举的属性创建 ref。如果要为可能还不存在的属性创建 ref，请改用 [`toRef`](https://cn.vuejs.org/api/reactivity-utilities.html#toref)。



### 3、生命周期钩子

setup()钩子函数是在beforeCreate()之前执行的。组合式API中不提供beforeCreate和created对应的函数了。

```js
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>生命周期</title>
</head>

<body>
  <div id="app">
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp } = Vue

  const app = createApp({
      
    setup() {
      console.log("setup");
    },
      
    beforeCreate(){
      console.log("beforeCreate");
    },
    created(){
      console.log("created");
    },
    
    beforeMount(){
      console.log("beforeMount");
    },
    
    mounted(){
      console.log("mounted");
    }    
  })

  app.mount('#app')
</script>

</html>
```

>执行顺序：
>
>setup
>beforeCreate
>created
>beforeMount
>mounted
>
>



在setup函数里面如何使用生命周期钩子函数（组合式api）：

#### 1）、 onBeforeMount()

注册一个钩子，在组件被挂载之前被调用。

当这个钩子被调用时，组件已经完成了其响应式状态的设置，但还没有创建 DOM 节点。它即将首次执行 DOM 渲染过程。



#### 2）、 onMounted()

注册一个回调函数，在**组件挂载完成后**执行。

组件在以下情况下被视为已挂载：

- 其所有同步子组件都已经被挂载 (不包含异步组件或 `<Suspense>` 树内的组件)。
- 其自身的 DOM 树已经创建完成并插入了父容器中。注意仅当根容器在文档中时，才可以保证组件 DOM 树也在文档中。

这个钩子通常用于执行需要访问组件所渲染的 DOM 树相关的副作用，或是在[服务端渲染应用](https://cn.vuejs.org/guide/scaling-up/ssr.html)中用于确保 DOM 相关代码仅在客户端执行。



#### 3）、 onBeforeUpdate()

注册一个钩子，在组件即将因为响应式状态变更而更新其 DOM 树之前调用。

这个钩子可以用来在 Vue 更新 DOM 之前访问 DOM 状态。在这个钩子中更改状态也是安全的。



#### 4）、 onUpdated()

注册一个回调函数，在组件因为响应式**状态变更而更新其 DOM 树之后调用**。

父组件的更新钩子将在其子组件的更新钩子之后调用。

这个钩子会在组件的任意 DOM 更新后被调用，这些更新可能是由不同的状态变更导致的。如果你需要在某个特定的状态更改后访问更新后的 DOM，请使用 [nextTick()](https://cn.vuejs.org/api/general.html#nexttick) 作为替代。



#### 5）、 onBeforeUnmount()

注册一个钩子，在**组件实例被卸载之前调用**。vue2中是beforeDestory。

当这个钩子被调用时，组件实例依然还保有全部的功能。



#### 6）、 onUnmounted() 

注册一个回调函数，在**组件实例被卸载之后调用**。vue2中是destoryed。

一个组件在以下情况下被视为已卸载：

- 其所有子组件都已经被卸载。
- 所有相关的响应式作用 (渲染作用以及 `setup()` 时创建的计算属性和侦听器) 都已经停止。

可以在这个钩子中手动清理一些副作用，例如计时器、DOM 事件监听器或者与服务器的连接。



### 4、依赖注入

#### 1）、 provide()

提供一个值，可以被后代组件注入。

`provide()` 接受两个参数：第一个参数是要注入的 key，可以是一个字符串或者一个 symbol，第二个参数是要注入的值。

> 与注册生命周期钩子的 API 类似，`provide()` 必须在组件的 `setup()` 阶段同步调用。

#### 2）、 inject()

**功能：**注入一个由祖先组件或整个应用 (通过 `app.provide()`) 提供的值。

**第一个参数：**注入的 key。Vue 会遍历父组件链，通过匹配 key 来确定所提供的值。如果父组件链上多个组件对同一个 key 提供了值，那么离得更近的组件将会“覆盖”链上更远的组件所提供的值。如果没有能通过 key 匹配到值，`inject()` 将返回 `undefined`，除非提供了一个默认值。

**第二个参数：**可选的，即在没有匹配到 key 时使用的默认值。它也可以是一个工厂函数，用来返回某些创建起来比较复杂的值。如果默认值本身就是一个函数，那么你必须将 `false` 作为第三个参数传入，表明这个函数就是默认值，而不是一个工厂函数。

与注册生命周期钩子的 API 类似，`inject()` 必须在组件的 `setup()` 阶段同步调用。

```js
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>provide_inject</title>
</head>
<body>
  <div id="app">
    <button @click="count++">点击{{ count }}次</button>
    <hr/>
    <my-parent></my-parent>
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, provide, inject } = Vue

  const Child = {
    template:  `
      <div>
         我是子组件 <button @click="add">加10</button>： {{ test }} - {{ fn() }}
      </div>
    `,
    setup () {
      // 注入值的默认方式
      const add = inject('add')
      // 如果没有提供test，那么，默认值是“测试”。
      const test = inject('test', '测试')
      // 如果默认值为函数，设置第三个参数为false
      const fn = inject('fn', () => {
        console.log(100000)
        return "函数"
      }, false)

      return {
        add,
        test,
        fn
      }
    }
  }

  const Parent = {
    template: `
              <div>
                  我是父组件count - {{ count }}
                  <hr/>
                  <my-child></my-child>
              </div>
    `,
    components: {
      MyChild: Child
    },
    setup () {
      const count = inject('count');
      return {
        count
      }
    }
  }

  const app = createApp({
    components: {
      MyParent: Parent
    },
    setup () {
      const count = ref(10)

      const add = () => {
        count.value += 10
      }
      // 提供值，可以是数据，也可以是函数
      provide('count', count)
      provide('add', add)
      provide('test',"hi")

      return {
        count
      }

    }
  })

  app.mount('#app')
</script>
</html>
```





### 5、组合式函数(自定义hooks)

\- 什么是“组合式函数”？
\- 如何定义和使用组合式函数
\- 异步状态
\- 约定和最佳实践

组合式函数和普通函数的区别：

1、 组合式函数里：可以使用组件相关的特性（生命周期：onMounted，onBeforeUpdate等），命名用use开头

2、普通函数里：不能使用组件相关的特性。



hooks：

​    钩子，钩什么东西？钩子钩的是vue组件的相关特性（如：生命周期，响应式等等）

#### 1）、 什么是“组合式函数”

在 Vue 应用的概念中，“组合式函数”(Composables) 是一个利用 Vue 的组合式 API 来封装和复用**有状态逻辑**的函数。

> 也可以叫做自定义hooks（钩子）

当构建前端应用时，我们常常需要复用公共任务的逻辑。例如为了在不同地方格式化时间，我们可能会抽取一个可复用的日期格式化函数。这个函数封装了**无状态的逻辑（跟组件的相关特性没有关系）**：它在接收一些输入后立刻返回所期望的输出。

相比之下，有状态逻辑（跟组件的特性有关）负责管理会随时间而变化的状态。一个简单的例子是跟踪当前鼠标在页面中的位置。在实际应用中，也可能是像触摸手势或与数据库的连接状态这样的更复杂的逻辑。

如果我们要直接在组件中使用组合式 API 实现鼠标跟踪功能，它会是这样的

`示例`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>composition_mouse</title>
</head>
<body>
  <div id="app">
    鼠标的位置为：({{ x }}, {{ y }})
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, onMounted, onUnmounted } = Vue

  const app = createApp({
    setup () {
      const x = ref(0)
      const y = ref(0)

      function updatePosition (event) {
        x.value = event.pageX
        y.value = event.pageY
      }

      onMounted(() => {
        window.addEventListener('mousemove', updatePosition, false)
      }) 
        
      onUnmounted(() => {
        window.removeEventListener('mousemove', updatePosition, false)
      })
        
      return {
        x, y
      }

    }
  })

  app.mount('#app')
</script>
</html>
```

> 但是，如果我们想在多个组件中复用这个相同的逻辑呢？我们可以把这个逻辑以一个组合式函数的形式提取到外部文件中

#### 2）、 如何定义和使用组合式函数

`示例`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>composition_mouse_hooks</title>
</head>
<body>
  <div id="app">
    鼠标的位置为：({{ x }}, {{ y }})
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, onMounted, onUnmounted } = Vue
  function useMouse () {
    const x = ref(0)
    const y = ref(0)

    function updatePosition (event) {
      x.value = event.pageX
      y.value = event.pageY
    }

    onMounted(() => {
      window.addEventListener('mousemove', updatePosition, false)
    }) 
    onUnmounted(() => {
      window.removeEventListener('mousemove', updatePosition, false)
    })
    return { x, y }
  }
    
  const app = createApp({
    setup () {
      const { x, y } = useMouse()
      return {
        x, y
      }

    }
  })

  app.mount('#app')
</script>
</html>
```

如你所见，核心逻辑完全一致，我们做的只是把它移到一个外部函数中去，并返回需要暴露的状态。和在组件中一样，你也可以在组合式函数中使用所有的[组合式 API](https://cn.vuejs.org/api/#composition-api)。现在，`useMouse()` 的功能可以在任何组件中轻易复用了。

更酷的是，你还可以嵌套多个组合式函数：一个组合式函数可以调用一个或多个其他的组合式函数。

示例`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>composition_mouse_more_hooks</title>
</head>
<body>
  <div id="app">
    鼠标的位置为：({{ x }}, {{ y }})
  </div>
</body>
<script src="./js/vue.global.js"></script>
<script>
  const { createApp, ref, onMounted, onUnmounted } = Vue
  
  function useEventListener (target, event, callback) {
    onMounted(() => {
      target.addEventListener(event, callback, false)
    }) 
    onUnmounted(() => {
      target.removeEventListener(event, callback, false)
    })
  }
    

  function useMouse () {
    const x = ref(0)
    const y = ref(0)

    // function updatePosition (event) {
    //   x.value = event.pageX
    //   y.value = event.pageY
    // }
    useEventListener(window, 'mousemove', (event) => {
      x.value = event.pageX
      y.value = event.pageY
    })

    
    return { x, y }
  }
  const app = createApp({
    setup () {
      const { x, y } = useMouse()
      return {
        x, y
      }

    }
  })

  app.mount('#app')
</script>
</html>
```

#### 3）、 异步状态

在做异步数据请求时，我们常常需要处理不同的状态：加载中、加载成功和加载失败。

`示例`

```html
<script setup>
    import {ref} from "vue";
    import axios from "axios";

    let data = ref(null);

    function getData(){
        axios({
            url:"http://121.89.205.189:3001/api/pro/list"
        })
        .then(res=>{
            data.value = res.data;
            console.log("data",data);
        })
    }

    getData();

</script>

<template>

  <div>
    <h1>异步组件加载的状态封装：</h1>
    <ul v-if="data">
        数据显示
        <li v-for="item in data.data" :key="item.proid">
            <p>商品编号：{{item.proid}}</p>
            <p>商品：{{item.proname}}</p>
        </li>
    </ul>
    <div v-else>加载中………………</div>

  </div>

</template>

<style scoped>


</style>
```

如果在每个需要获取数据的组件中都要重复这种模式，那就太繁琐了。让我们把它抽取成一个组合式函数：

`异步hooks示例：`

```html
<script setup>
import { reactive, ref } from "vue";
import axios from "axios";

function useData(url) {
  let data = ref(null);
  let err = ref(null);

  axios({
    url,
  })
    .then((res) => {
      data.value = res.data;
      console.log("data", data);
    })
    .catch((err) => {
      err.value = err;
    });

  return { data, err };
}

const { data, error } = useData("http://121.89.205.189:3001/api/pro/list");


</script>

<template>
  <div>
    <h1>异步组件加载的状态封装：</h1>
    <ul v-if="data">
      数据显示
      <li v-for="item in data.data" :key="item.proid">
        <p>商品编号：{{ item.proid }}</p>
        <p>商品名称：{{ item.proname }}</p>
      </li>
    </ul>
    <div v-else>加载中………………</div>
  </div>
</template>

<style scoped>
</style>
```

`useData()` 接收一个静态的 URL 字符串作为输入，所以它只执行一次请求，然后就完成了。但如果我们想让它在每次 URL 变化时都重新请求呢？那我们可以让它同时允许接收 ref 作为参数：

`接收不同url的异步hooks示例：`

```html
<script setup>
import { isRef, ref, watchEffect, unref, computed } from "vue";
import axios from "axios";

function useData(url) {
  console.log("useData");

  // 2.1、data和err都是响应式的数据
  let data = ref(null);
  let err = ref(null);

  if (isRef(url)) {//如果url是ref的对象的话，那么就使用watchEffect。
    //2.2、 定义了一个监听。当url发生变化时，就会调用回调函数。
    watchEffect(() => {
      console.log("发送请求获取数据");
      axios({
        url: unref(url), //unref() 解包可能为 ref 的值  unref(url) <===> url.value
      })
        .then((res) => {
          data.value = res.data;
        })
        .catch((err) => {
          err.value = err;
        });
    });

  } else {
    // 如果url不是ref的对象的话，就不用使用watchEffect，直接发送请求即可。

  }

  return { data, err };
}

let pageIndex = ref(1);
// 1、定义计算属性url，当pageInde的值发生变化时，url就会发生变化，url是响应式的数据。
const url = computed(
  () => "http://121.89.205.189:3001/api/pro/list?count=" + pageIndex.value
);

const { data, error } = useData(url);

</script>

<template>
  <div>
    <h1>异步组件加载的状态封装：</h1>
    <p>选择页码：{{ pageIndex }}</p>
    <select v-model="pageIndex">
      <option value="1">1</option>
      <option value="2">2</option>
      <option value="3">3</option>
      <option value="4">4</option>
      <option value="5">5</option>
    </select>
    <ul v-if="data">
      数据显示
      <li v-for="item in data.data" :key="item.proid">
        <p>商品编号：{{ item.proid }}</p>
        <p>商品名称：{{ item.proname }}</p>
      </li>
    </ul>
    <div v-else>加载中………………</div>
  </div>
</template>

<style scoped>
</style>
```



#### 4）、 约定和最佳实践

##### 1）、 命名

组合式函数约定用驼峰命名法命名，并以“use”作为开头。

##### 2）、 输入参数

尽管其响应性不依赖 ref，组合式函数仍可接收 ref 参数。如果编写的组合式函数会被其他开发者使用，你最好在处理输入参数时兼容 ref 而不只是原始的值。[`unref()`](https://cn.vuejs.org/api/reactivity-utilities.html#unref) 工具函数会对此非常有帮助：

```js
import { unref } from 'vue'

function useFeature(maybeRef) {
  // 若 maybeRef 确实是一个 ref，它的 .value 会被返回
  // 否则，maybeRef 会被原样返回
  const value = unref(maybeRef)
}
```

如果你的组合式函数在接收 ref 为参数时会产生响应式 effect，请确保使用 `watch()` 显式地监听此 ref，或者在 `watchEffect()` 中调用 `unref()` 来进行正确的追踪。

##### 3）、  返回值

你可能已经注意到了，我们一直在组合式函数中使用 `ref()` 而不是 `reactive()`。我们推荐的约定是组合式函数始终返回一个包含多个 ref 的普通的非响应式对象，这样该对象在组件中被解构为 ref 之后仍可以保持响应性：

```js
// x 和 y 是两个 ref
const { x, y } = useMouse()
```

从组合式函数返回一个响应式对象会导致在对象解构过程中丢失与组合式函数内状态的响应性连接。与之相反，ref 则可以维持这一响应性连接。

如果你更希望以对象属性的形式来使用组合式函数中返回的状态，你可以将返回的对象用 `reactive()` 包装一次，这样其中的 ref 会被自动解包，例如：

```js
const mouse = reactive(useMouse())
// mouse.x 链接到了原来的 x ref
console.log(mouse.x)
```

```js
Mouse position is at: {{ mouse.x }}, {{ mouse.y }}
```

##### 4）、  副作用

在组合式函数中的确可以执行副作用 (例如：添加 DOM 事件监听器或者请求数据)，但请注意以下规则：

- 如果你的应用用到了[服务端渲染](https://cn.vuejs.org/guide/scaling-up/ssr.html) (SSR)，请确保在组件挂载后才调用的生命周期钩子中执行 DOM 相关的副作用，例如：`onMounted()`。这些钩子仅会在浏览器中被调用，因此可以确保能访问到 DOM。
- 确保在 `onUnmounted()` 时清理副作用。举例来说，如果一个组合式函数设置了一个事件监听器，它就应该在 `onUnmounted()` 中被移除 (就像我们在 `useMouse()` 示例中看到的一样)。当然也可以像之前的 `useEventListener()` 示例那样，使用一个组合式函数来自动帮你做这些事。

##### 5）、  使用限制

组合式函数在 `<script setup>` 或 `setup()` 钩子中，应始终被**同步地**调用。在某些场景下，你也可以在像 `onMounted()` 这样的生命周期钩子中使用他们。

这个限制是为了让 Vue 能够确定当前正在被执行的到底是哪个组件实例，只有能确认当前组件实例，才能够：

1. 将生命周期钩子注册到该组件实例上

2. 将计算属性和监听器注册到该组件实例上，以便在该组件被卸载时停止监听，避免内存泄漏。

   

## 4、补充：总结组合式api和选项式api的对比

1）、选项式api限定了同一个业务逻辑中不同选项（methods，watch，生命周期等等），只能放在不同的选项（属性）里。



2）、而组合式api把不同选项单独写成函数，在处理同一个业务逻辑时，如下需要用到哪个选项，直接调用对应的函数就行。而且组合式api还可以把不同的业务单独封装出来。这样的话，组件里的代码就会更少，结构更加清晰。



​    如：完成一个业务，需要在methods、watch、mounted等选项里写代码，那么，组合式api，只需要定义函数，使用watch函数，onMounted函数就行。这样的话，就可以把完成不同业务的代码放在一起。



![image-20221015114542932](.\\image-20221015114542932.png)



## 5、补充： 响应式的进一步理解：

 1）、响应式的数据（ref，reactive，computed），在模板上使用，不用担心响应式的问题（因为，vue自动做了订阅）

 2）、响应式的数据（ref，reactive，computed），在js中使用，就需要程序员自己完成响应式的处理（如：watch，watchEffect）

```js
<template>
    <div>
      <h1>（提升认识）响应式的进一步理解：</h1>
      <!-- 1、模板上：当响应式的数据发生变化时，模板会自动重新执行 -->
      <p>count:{{count}}</p>
      <input type="button" value="修改count" @click="changeCount">
      <p>s:{{s}}</p>

    </div>
  </template>
    
  <script setup>

  // 响应式的进一步理解：
  // 1、响应式的数据（ref，reactive，computed），在模板上使用，不用担心响应式的问题（因为，vue自动做了订阅）
  // 2、响应式的数据（ref，reactive，computed），在js中使用，就需要程序员自己完成响应式的处理（如：watch，watchEffect）

  import {ref, watchEffect,watch,computed} from "vue";

  let count = ref(2);

  function changeCount(){
    count.value++;
  }

  // 2、在js中，当响应式的数据发生变化时，js代码不会自动执行。所以说，vue中提供了watch，watchEffect让程序员在js代码时，能够完成响应式数据引起的js代码的执行
  function fn(){
    console.log("fn:count.value",count.value);
  }

  watchEffect(()=>{
    console.log("watchEffect:count.value",count.value);
  })

  watch(count,()=>{
    console.log("watch:count.value",count.value);
  });

// 定义一个计算属性，计算属性和ref（reactive）是同样的道理，都是定义数据。
  let s = computed(()=>{
    console.log("computed:count.value",count.value);
    return  count.value++;
  })

  
  </script>
  
  <style scoped>
  
  </style>
```



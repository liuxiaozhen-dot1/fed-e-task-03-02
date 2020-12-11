##### 1.简述Vue首次渲染过程

1. 在首次进行渲染之前，首先进行vue初始化实例成员和静态成员。

2. 初始化完成后，调用vue构造函数new Vue()，在构造函数中调用_init()方法。

3. 在_init()方法中，最终调用$mount()

4. vm.$mount()是src/platform/web/entry-runtime-with-compiler.js中定义的，核心作用是把模板编译为render函数，判断是否有render选项，如果没有，则会获取template选项，如果template也没有，会把el中的内容作为模板，通过compileToFunctions()方法将模板编译为render函数，编译好以后，将render存入到options.render中。

5. 调用src/platforms/web/runtime/index.js文件中的$mount方法,这个方法中会重新获取el，因为如果是运行时版本的话，是不会走entry-runtime-with-compiler.js这个入口中获取el，所以如果是运行时版本的话，我们会在runtime/index.js的$mount()中重新获取el。

6. 调用src/platforms/web/runtime/index.js文件中的$mount方法,这个方法中会重新获取el，因为如果是运行时版本的话，是不会走entry-runtime-with-compiler.js这个入口中获取el，所以如果是运行时版本的话，我们会在runtime/index.js的$mount()中重新获取el。

7. 创建完watcher，会调用一次get，在get方法中会调用updateComponent(),updateComponent会调用实例化时传入的render（）或者是编译模板以后生成的render（），返回vnode。然后调用vm._update（），调用vm.__patch__方法，将虚拟dom转化为真实dom并挂载到页面上，将生成的真实dom记录到vm.$el()中

8. 挂载结束，最终返回Vue实例。

##### 2、请简述 Vue 响应式原理

1. 在init()方法中先调用initState()初始化vue实例状态，在initState方法中调用initData(),initData()把data属性注入到vue实例上，并且调用observe(data)将data对象转化成响应式的对象。

2. observe是响应式的入口，在observe(value)中，首先判断传入的参数value是否有对象，如果不是对象直接返回，再判断value对象是否有_ob_这个属性，如果有说明做过了响应式原理，则直接返回。如果没有，创建observe对象并且返回。

3. 在创建observer对象时，给当前的value对象定义不可枚举的__ob__属性，记录当前的observer对象，然后再进行数组的响应式处理和对象的响应式处理，数组的响应式处理就是拦截数组的几个特殊的方法，push、pop、shift等，然后找到数组对象中的__ob__对象中的dep,调用dep的notify()方法，再遍历数组中每一个成员，对每个成员调用observer()，如果这个成员是对象的话，也会转换成响应式对象。对象的响应式处理，就是调用walk方法，walk方法就是遍历对象的每一个属性，对每个属性调用defineReactive方法

4. defineReactive会为每一个属性创建对应的dep对象，让dep去收集依赖，如果当前属性的值是对象，会调用observe。defineReactive中最核心的方法是getter 和 setter。getter 的作用是收集依赖，收集依赖时, 为每一个属性收集依赖，如果这个属性的值是对象，那也要为子对象收集依赖，最后返回属性的值。在setter 中，先保存新值，如果新值是对象，也要调用 observe ，把新设置的对象也转换成响应式的对象,然后派发更新（发送通知）调用dep.notify()

5. 收集依赖时，在watcher对象的get方法中调用pushTarget,记录Dep.target属性，访问data中的成员的时候收集依赖，defineReactive的getter中收集依赖，把属性对应的 watcher 对象添加到dep的subs数组中，给childOb收集依赖，目的是子对象添加和删除成员时发送通知。

6. 在数据发生变化的时候，会调用dep.notify()发送通知，dep.notify()会调用watcher对象的update()方法，update()中的调用的queueWatcher()会去判断watcher是否被处理，如果这个watcher对象没有的话添加到queue队列中，并调用flushScheduleQueue()，flushScheduleQueue()触发beforeUpdate钩子函数调用watcher.run()：run()-->get() --> getter() --> updateComponent()

7. 然后清空上一次的依赖

8. 触发actived的钩子函数

9. 触发actived的钩子函数


##### 3、请简述虚拟 DOM 中 Key 的作用和好处

v-for遍历的时候，能够追踪每个节点的身份，在进行新旧虚拟DOM节点比较的时候，会基于key的变化重新排列元素的顺序，从而重用和重新排序现有的元素，并且移除key不存在的元素，方便在diff过程中找到对应的节点，然后复用，从而减少dom的操作。

##### 4、请简述 Vue 中模板编译的过程

模板编译的入库是compileToFunctions,先从缓存中加载编译好的render函数，如果没有就去调用compile函数合并选项，然后调用baseCompile(参数：合并好的选项)编译模板。之后通过调用createFunction函数，把baseCompile中生成的字符串形式JS代码转化为函数形式。当render和staticRenderFns初始化完毕，挂载到Vue实例的options对应的属性上




#### 笔记
1. 
el不能是body或者是html标签
如果没有render，把template转换成render函数
如果有render方法，直接调用mount挂载DOM
$mount是在init方法中调用的

2. 使用抽象语法树
模板字符串转换成AST后，可以通过AST对模板做优化处理
标记模板中的静态内容，在patch的时候直接掉过静态内容
在patch的过程中静态内容不需要对比和重新渲染


3.  组件的patch过程

先创建父组件，再创建子组件。
先挂载子组件，再挂载父组件。
在 createElement() 函数中调用 createComponent() 创建的是组件的 VNode。组件对象是在组件的 init 钩子函数中创建的，然后在 patch() --> createElm() --> createComponent() 中挂载组件
全局组件之所以可以在任意组件中使用是因为 Vue 构造函数的选项被合并到了 VueComponent 组件构造函数的选项中。
局部组件的使用范围被限制在当前组件内是因为，在创建当前组件的过程中传入的局部组件选项，其它位置无法访问

4. 使用虚拟DOM
避免直接操作DOM，提高开发效率
作为一个中间层可以跨平台
虚拟DOM不一定可以提高性能
首次渲染的时候会增加开销
复杂视图情况下提升渲染性能

5. 
异步更新队列，$nextTick
Vue更新DOM是异步执行的，批量的
在下次DOM更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的DOM
mounted中的更新数据是一个异步的过程，可以通过nextTick获取最新值，在nextTick的回调函数中，视图已经更新完毕，所以可以获取视图上的最新数据。
用户传入的回调函数会存到callbacks数组中，然后在timerFunc函数中以微任务的形式执行callbacks。微任务是在本次任务完成之后，才会去执行。nextTick是获取DOM上的最新数据，当微任务执行的时候，DOM元素还未渲染到浏览器上，但其实在nextTick中的回调函数执行之前，数据已经被改变了，当数据改变的时候，会通知watcher渲染视图，但在watcher里是先更新DOM树，而什么时候将DOM数据更新到浏览器上，是这次事件循环结束之后，才会执行DOM的渲染。nextTick中获取DOM数据是从DOM树上获取数据的，此时DOM还未渲染到浏览器中。nextTick中优先使用Promise执行微任务。在非IE浏览器中，使用了MutationObserver执行微任务。如果Promise和MutationObserver都不支持，则使用setImmediate，setImmediate只有IE浏览器和Nodejs支持。有的浏览器不支持微任务，则降级使用setTimeout

6. 
首次渲染过程
Vue构造函数
this._init()
this.$mount()
mountComponent()
new Watcher() 渲染Watcher
updateComponent()
vm.render() -> createElement()
vm._update()
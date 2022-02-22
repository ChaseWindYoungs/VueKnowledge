### ***Vue3.0 面试题总结***

1. **响应式**
   - vue2
     数组和对象类型的值变化的时候，通过`defineReactive`方法，借助了`defineProperty`，将所有的属性添加了`getter`和`setter`。用户在取值和设置的时候，可以进行一些操作。
     缺陷：只能监控最外层的属性，如果是多层的，就要进行全量递归。
     并不能检测对象属性的添加和删除。直接造成了数组元素的直接修改不会触发响应式机制，
     get 里面会做依赖搜集（dep[watcher, watcher]）
     set 里面会做数据更新（notify，通知 watcher 更新）
     ```jsx
     const obj = {};
     Object.defineProperty(obj, 'text', {
         get: function() {
             console.log('get val');&emsp;
         },
         set: function(newVal) {
             console.log('set val:' + newVal);
             document.getElementById('input').value = newVal;
             document.getElementById('span').innerHTML = newVal;
         }
     });
     const input = document.getElementById('input');
     input.addEventListener('keyup', function(e){
         obj.text = e.target.value;
     })
     ```
   - vue3
     Proxy API 做数据劫持，它劫持的是整个对象，自然对于对象的属性的增加和删除都能检测到，
     通过 Proxy 实现双向响应式绑定，相比 defineProperty 的遍历属性的方式效率更高，性能更好，另外 Virtual DOM 更新只 diff 动态部分、事件缓存等，也带来了性能上的提升
     ```jsx
     const input = document.getElementById("input");
     const p = document.getElementById("p");
     const obj = {};
     const newObj = new Proxy(obj, {
       get: function (target, key, receiver) {
         console.log(`getting ${key}!`);
         return Reflect.get(target, key, receiver);
       },
       set: function (target, key, value, receiver) {
         console.log(target, key, value, receiver);
         if (key === "text") {
           input.value = value;
           p.innerHTML = value;
         }
         return Reflect.set(target, key, value, receiver);
       },
     });
     input.addEventListener("keyup", function (e) {
       newObj.text = e.target.value;
     });
     ```
2. **Tree-Shaking Support（摇树优化）**
   - tree-sharking 即在构建工具构建后消除程序中无用的代码，来减少包的体积
     相比 Vue2.x 导入整个 Vue 对象，Vue3.0 支持按需导入，只打包需要的代码。
   - Tree-Shaking 依赖 ES2015 模块语法的静态结构（即 import 和 export），通过编译阶段的静态分析，找到没有引入的模块并打上标记。
   - 在项目中如果没有引入 Transition、KeepAlive 等不常用的组件，那么它们对应的代码就不会打包进去。
3. **组合式 API**
   - 选项式 API 存在的缺陷
     随着业务复杂度越来越高，代码量会不断的加大；由于代码需要遵循 option 的配置写到特定的区域，导致后续维护非常的复杂，代码可复用性也不高。比如，很长的 methods 区域代码、data 变量声明与方法区域未在一起。
   - 与 mixins 的比较
     对于上述提到的问题，我们可能会想到会 mixins 来解决，但是当抽离并引用了大量的 mixins，
     - 问题：
       - 命名冲突，数据来源不清晰。
     - 使用场景：
       - Vue 的 mixin 的作用就是抽离公共的业务逻辑，原理类似对象的继承，当组件初始化的时候，会调用 mergeOptions 方法进行合并，采用策略模式针对不同的属性进行合并。 如果混入的数据和本身组件的数据有冲突，采用本身的数据为准。
   - 组合式 API 和 mixins 二者的差别
     - 层级不同：组合式 API 与组件是嵌套关系，而 mixin 与组件是同 层级关系
     - 影响面不同：组合式 API 作为组件的被调用方，并且变量逻辑是组件控制，耦合性很低。而 mixins 是耦合在代码逻辑里，并且存在变量的互相引用，为将来的升级和维护埋下隐患。
4. 源码优化
   - 使用 monorepo 来管理源码
     Vue.js 2.x 的源码托管在 src 目录，然后依据功能拆分 出了
     compiler（模板编译的相关代码）、
     core（与平台无关的通用运行时代码）、
     platforms（平台专有代码）、
     server（服务端渲染的相关代码）、
     sfc（.vue 单文件解析相关代码）、
     shared（共享工具代码）等目录。
     Vue.js 3.0，整个源码是通过 monorepo 的方式维护的，根据功能将不同的模块拆分到 packages 目录下面不同的子目录中，每个 package 有各自的 API、类型定义和测试。
   - 使用 Typescript 来开发源码
     - Vue.js 2.x 选用 Flow 做类型检查，来避免一些因类型问题导致的错误，但是 Flow 对于一些复杂场景类型的检查，支持得并不好。
     - Vue.js 3.0 抛弃了 Flow ，使用 TypeScript 重构了整个项目。TypeScript 提供了更好的类型检查，能支持复杂的类型推导；由于源码就使用 TypeScript 编写，也省去了单独维护 d.ts 文件的麻烦。
5. Vue 生命周期钩子是如何实现的

   Vue 的生命周期钩子是回调函数，当创建组件实例的过程中会调用相应的钩子方法。 内部会对钩子进行处理，将钩子函数维护成数组的形式。

6. 组件生命周期（详细）
   - 生命周期
     - beforeCreate 在实例初始化之后，数据观测 observer 和 event、watcher 事件配置之前被调用
     - created 实例已经创建完成，在这一步，以下配置被完成
       - 数据观测
       - 属性和方法的运算
       - watch/event 时间回调
       - $el 尚未生成
     - beforeMount 在挂载之前被调用，render 尚未被调用
     - mounted el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用
     - beforeUpdate 数据更新时，被调用，发生在虚拟 Dom 重新渲染和打补丁之前
     - update 由于数据更改导致的虚拟 Dom 重新渲染和打补丁，在这之后调用
     - activited keep-alive 专属，组件被**激活**时调用
     - deactivated keep-alive 专属，组件被**销毁**时调用
     - beforeDestroy 实例销毁之前调用
     - destroyed 实例销毁之后调用，调用后 Vue 实例的所有东西都会被解绑，所有的事件监听会被移除，子实例被销毁，该钩子在服务端渲染期间不被调用 keep-alive（activated & deactivated）
     ![Vue生命周期详解](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/8/19/16ca74f183827f46~tplv-t2oaga2asx-watermark.awebp)
7. Vue 的组件 data 为什么必须是一个函数?

   `new Vue`是一个单例模式，不会有任何的合并操作，所以根实例不必校验 data 一定是一个函数。
   **组件的 data 必须是一个函数**，是为了防止两个组件的数据产生污染。
   如果都是对象的话，会在合并的时候，指向同一个地址。
   而如果是函数的时候，合并的时候调用，会产生两个空间。

8. nextTick 的原理
   - 微任务
     - nextTick 中的回调是在 `下次Dom更新循环结束之后` 执行的延迟回调
     - 可以用于获取更新后的 Dom
     - Vue 中的数据更新是异步的，使用 nextTick 可以保证用户定义的逻辑在更新之后执行
9. Vue.set 方法（Vue2）
   - vue 给对象和数组本身都增加了 dep 属性
   - 当给对象新增不存在的属性的时候，就会触发对象依赖的 watcher 去更新
   - 当修改数组索引的时候，就调用数组本身的 splice 方法去更新数组
10. Vue中的设计模式(实际场景中的应用)   
    [设计模式在vue中的应用](https://juejin.cn/post/6844903760674701320)
   - 外观模式
     - 根据类似逻辑和布局的父组件，将内容进行抽取，内部的组成用子组件构建，让外部父组件引用即可
  
   - 工厂模式
     - 多个相同子组件，因参数不同，在同一个组件中，可根据工厂模式，传递不同的参数，动态生成
       - eg： 根据不同类型，匹配不同数据，配合动态组建，生成组件

   - 状态模式
     - 首先定义环境，在环境中规定好,该状态下执行的逻辑，以及该状态下渲染的内容，再初始化一个最开始的状态
     - 在每一个状态中，执行当前状态的逻辑，再将要改变的时候，通知环境，执行下一步的更改，将更改传递给环境，由环境去执行下一步的渲染和逻辑
     - 环境只关心状态的改变通知后触发的效果，状态内部去处理逻辑
     - 因为状态的复杂程度不可知，如果存在回到上一个状态的时候，因该在环境中定义变量存储每一次改变的状态的顺序，无论状态触发或者环境触发，都可不污染每个状态的逻辑去实现

   - 模板方法模式
     - [模板方法模式](https://juejin.cn/post/6844903775669321736)
     - 模板方法模式在一个方法中定义一个算法的骨架，而将一些步骤的实现延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中某些步骤的具体实现
     - Vue中slot的设计模式理念,是按照模板方法模式来实现的，React 的 render也是如此

   - 享元模式
     - 享元模式使用共享技术实现相同或者相似对象的重用。也就是说实现相同或者相似对象的代码共享。
     - 一些代码，可以共用的，先配置为默认值，在扩展的方法上，去动态的传递该方法所特有的默认值
     - 当调用扩展方法时，传递进来的新的值与扩展方法的默认值，合并，用传递来的值强制替换掉扩展方法上的属性值
     - 再传递给初始对象进行构建
     - message，model，drawer 都是这种类型，可以提供扩展，方法也类似，通过传递值的变化，共享基础代码
  
   - 观察者模式
     - 观察者（Observer）模式与发布（Publish）/订阅（Subscribe）的关系，他们的行为方式相似add，notify/trigger，最明显的区别是观察者add一个具体观察者对象，发布/订阅add一个订阅事件
     - 观察者模式
       - 比较概念的解释是，目标和观察者是基类，目标提供维护观察者的一系列方法，观察者提供更新接口。具体观察者和具体目标继承各自的基类，然后具体观察者把自己注册到具体目标里，在具体目标发生变化时候，调度观察者的更新方法。
       - 比如有个“天气中心”的具体目标A，专门监听天气变化，而有个显示天气的界面的观察者B，B就把自己注册到A里，当A触发天气变化，就调度B的更新方法，并带上自己的上下文。
       - [观察者模式](https://images2015.cnblogs.com/blog/555379/201603/555379-20160313183429007-1351424959.png)
     - 发布/订阅模式
       - 比较概念的解释是，订阅者把自己想订阅的事件注册到调度中心，当该事件触发时候，发布者发布该事件到调度中心（顺带上下文），由调度中心统一调度订阅者注册到调度中心的处理代码。
       - 比如有个界面是实时显示天气，它就订阅天气事件（注册到调度中心，包括处理程序），当天气变化时（定时获取数据），就作为发布者发布天气信息到调度中心，调度中心就调度订阅者的天气处理程序。
       - [发布/订阅模式](https://images2015.cnblogs.com/blog/555379/201603/555379-20160313183439366-1623019133.png)
     - 虽然两种模式都存在订阅者和发布者（具体观察者可认为是订阅者、具体目标可认为是发布者），但是观察者模式是由具体目标调度的，而发布/订阅模式是统一由调度中心调的，所以观察者模式的订阅者与发布者之间是存在依赖的，而发布/订阅模式则不会。
     - 在vue中体现在我们可以在组件上绑定ref去动态的获取子组件的接口以及数据，相当于子组件是观察者，父组件是订阅者，当子组件的数据发生变化时，可以触发或者接收到新的数据
  
   - 适配器模式
     - 适配器模式（有时候也称包装样式或者包装）将一个类的接口适配成用户所期待的。一个适配允许通常因为接口不兼容而不能在一起工作的类工作在一起，做法是将类自己的接口包裹在一个已存在的类中
     - 场景
       - 假设在项目中我们原本使用了Swiper这个组件，使用方式如下：
        ```js 
            <swiper :prop-x="px" :prop-y="py" :prop-z="pz" />
        ```
       - 后来又有了一个功能更强大，性能更好的滑动组件——NbSwiper，使用方式如下：
        ```js 
            <nb-swiper :prop-x="px" :prop-yy="pz" :prop-z="pw" />
        ```
      - 现在需求是要把项目中所有的Swiper组件替换为NbSwiper,但是NbSwiper的属性值有一部分与Swiper通用，还有一些不通用，还有其余的扩展的新的
      - 要解决这个问题我们可以选择将Swiper作为一个装饰组件
        ```js 
            // Swiper.vue
            // 项目原本使用的Swiper组件会被替换掉，我们自己封装一个Swiper组件
            <template>
              <!-- 进行转换 -->
              <nb-swiper :prop-x="propX" :prop-yy="propZ" :prop-z="propW" />
            </tempalte>
            <script>
              export default {
                props: {
                  // 接受原本Swiper的props和NbSwiper支持的props
                  propX: String,
                  propY: String,
                  propYy: String,
                  propZ: String,
                  propW: String,
                }
                ...
              }
            </script>
        ```
      - 现在我们的工作只需在代码中增加对NbSwiper的新属性值的兼容
  
  - 装饰者模式
    - 装饰模式指的是在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象
  
  - 代理模式
    - 所谓的代理者是指一个类别可以作为其它东西的接口。代理者可以作任何东西的接口：网上连接、存储器中的大对象、文件或其它昂贵或无法复制的资源。

<!-- axios 如何终端请求，取消重复请求 -->


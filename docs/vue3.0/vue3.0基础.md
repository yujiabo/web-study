###### 1. setup函数

vue单页面使用到的变量和方法都定义在setup函数中,return后才能被页面引用

```vue
export default {
   setup(){
       const name = '张三'
       const person = {name,age:30}
       function goWork(){
         consle.log('工作')
       }
       return {name,person,goWork}
   }
}
```

> 注意：直接定义的变量修改不会引起页面值的修改

###### 2. ref函数

使用ref包裹的变量或者对象会变成响应式数据，ref对象值的修改会引起页面值的修改，原始值不会被修改，主要用于基础数据的引用，本质是拷贝一份数据。

```
<template>
   <div @click="changeAge">{{refAge}}</div>
</template>
<script>
import {ref} from "vue"
export default {
       setup(){
          const age = 30
          const refAge = ref(age)
          function changeAge(){
             refAge.value = 50
          }
          return {refAge,changeAge}
       }
}
</script>
```

> 上面示例中调用changeAge函数后age的值不会变，refAge的值会变为50

###### 3. toRef函数

使用toRef包裹的对象会变成响应式数据，toRef对象值的修改不会引起页面值的变化，原始值会被修改，主要用于需要修值不需要刷新页面的场景，toRef接收两个参数：第一个参数是要操作的对象，第二个参数是对象的属性名

```
<template>
   <div @click="changeAge">{{person.name}}</div>
</template>
<script>
import {toRef} from "vue"
export default {
       setup(){
          const person = {name:'张三',age:30}
          const refNmae = toRef(person,'name')
          function changeAge(){
             refNmae.value = '李四'
          }
          return {changeAge,person}
       }
}
</script>
```

###### 3. reactive函数

使用reactive包裹的对象会变成响应式数据，reactive对象值的修改会引起页面值的修改,返回一个响应式的Proxy，主要用于复杂数据的引用如：对象、数组

```
<template>
   <div @click="changeAge">{{react.name}}</div>
</template>
<script>
import {reactive} from "vue"
export default {
       setup(){
          const person = {name:'张三',age:30}
          const react = reactive(person)
          function changeAge(){
             react.name = '李四'
          }
          return {changeAge,react}
       }
}
</script>
```

###### 4. toRefs函数

使用toRefs包裹的对象会变成响应式数据，toRefs对象值的修改不会引起页面值的变化，原始值会被修改，主要用于批量设置多个数据为响应数据

```
<template>
   <div @click="changeAge">{{name}}</div>
   <div>{{attribute.weight}}</div>
</template>
<script>
import {toRefs,reactive} from "vue"
export default {
       setup(){
          const person = {name:'张三',age:30,attribute:{weight:'80',height:170}}
          const refs = toRefs(reactive(person))
          function changeAge(){
             refs.name = '李四'
          }
          return {changeAge,...refs}
       }
}
</script>
```

> toRefs的对象配合扩展符(...)使用，toRefs传入的对象需要使用reactive包裹

###### 5. 计算属性

vue3.x使用computed组合式API实现, setup函数中可以有多个computed。

```
<template>
   <div >年龄总和:{{res}}</div>
   <div @click='modifyAction'>修改张三年龄</div>
</template>
<script>
import {reactive,computed} from "vue"
export default {
       setup(){
          const person1 = reactive({name:'张三',age:30,attribute:{weight:'80',height:170}})
          const person2 = reactive({name:'李四',age:50,attribute:{weight:'70',height:160}})
          function modifyAction (){
            person1.age = 40
          }
          var res =  computed(()=>{
              return person1.age + person2.age
          })
          return {res,modifyAction}
       }
}
</script>
```

> 计算结果 = computed(返回计算结果的函数)

###### 6. 侦听器

vue3.x使用watch组合式API实现，setup函数中可以有多个watch

```
<template>
   <div @click="res++">点击数字++:{{res}}</div>
   <div @click='modifyAction'>修改张三年龄</div>
</template>
<script>
import {ref,reactive,watch} from "vue"
export default {
       setup(){
          const res = ref(0)
          const person = reactive({name:'张三',age:{num:50}})
           // 监听对象数据的值
          watch(()=>person.age.num,(newVal,oldVal)=>{
               console.log("张三新年龄:",newVal)
               console.log("张三旧年龄:",oldVal)
          })
           // 监听基本数据的值
          watch(res,(newVal,oldVal)=>{
               console.log("数字新值:",newVal)
               console.log("数字旧值:",oldVal)
          })
           // 监听多个值  immediate：初始化时是否执行一次，默认false
          const res_a = ref(1)
          watch([res,res_a],(newVal,oldVal)=>{
            console.log("新的多个值:",newVal)
            console.log("旧的多个值:",oldVal)
          },{immediate:true})
          function modifyAction(){
            person.age.num = 20
          }
          return {res,modifyAction}
       }
}
</script>
```

> watch(监听属性,(newVal,oldVal)=>{ },{immediate:是否立即执行一次})

###### 7. 组件间传值

父组件传值使用provide，子组件收值使用inject

父组件代码：

```
<template>
   <div >父组件传值</div>
</template>
<script>
import {provide} from "vue"
export default {
       setup(){
          const person = reactive({name:'张三',age:30,attribute:{weight:'80',height:170}})
          // 第一个参数是Key，第二个参数是值
          provide('name',person)
       }
}
</script>
```

子组件代码

```
<template>
   <div >子组件收值</div>
</template>
<script>
import {inject} from "vue"
export default {
       setup(){
          const p1 = inject('name')
          console.log(p1)
       }
}
</script>
```

###### 8. vuex的使用

vuex在vue3.x中的使用基本和vue2.x中相同

- 创建数据仓库

```
import {createStore} from 'vuex'
const store = createStore({
     state:{
       // 数据名:数据值
       name:''
     },
     //同步调用方法
     mutations:{
       changeName(state,value){
          state.name = value
       }
     },
     // 异步调用方法，异步方法中必须调用同步方法才能修改state对象中的值
     actions:{
        modify(store,value){
           store.commit('changeName',value)
        }
     }
})
export default store
```

- vuex挂在到app上

```
// 在main.js文件中引入数据仓库对象store
import App from './App.vue'
import store from './store'
// vue3.x中挂在对象使用use(对象)的方式
createApp(App).use(store).mount('#app')
```

- 页面使用

```
<template>
   <div>{{name}}</div>
</template>
<script>
import {useStore} from "vuex"
export default {
       setup(){
          const store = useStore()
           // 同步修改值
          store.commit('changeName','李四')
          // 异步修改值
          store.dispatch('modify','张三')
          // 获取值
          const name = store.state.name
          console.log(name)
          return {name}
       }
}
</script>
```

###### 9. 生命周期函数

> onBeforeMount  ---  挂载开始之前被调用
>
> onMounted  ---  组件挂载时调用
>
> onBeforeUpdate  ---  数据更新时调用
>
> onUpdated  ---  数据更改导致虚拟DOM重新渲染之后会调用该函数
>
> onBeforeUnmont  ---  卸载组件实例之前调用
>
> onUnmounted  ---  卸载组件实例之后调用

页面使用

````
<template>
</template>
<script>
import {onBeforeMount} from "vue"
export default {
       setup(){
          onBeforeMount(()=>{
             // 生命周期函数回调方法
          })
       }
}
</script>
````

###### 10. 组合式API的抽离封装

上文介绍的2-6章节都是数据组合式api的内容，任何一个组合式api都可以单独抽离到另一个文件，然后在setup函数中使用。

第一步新建一个单独的js文件

````
// public.js文件
import {reactive,computed} from 'vue'
const public = ()=>{
    const res = reactive({
        name:'张三',
        age:'40'
    })
    return res
}
export default public
````

第二步导入js文件使用

```
<template>
   <div>{{res.name}}</div>
</template>
<script>
import public from "public.js"
export default {
       setup(){
         // 使用抽离封装的数据
         const res = public()
         return {res}
       }
}
</script>
```

#### 补充

Vue3.x比Vue2.x快，vue3.x去掉beforeCreate、create生命周期函数用setup函数代替，vue3.x代码和vue2.x代码可以在项目中同时存在。

 
``<keep-alive></keep-alive>``是vue内置的组件，主要用于保留组件状态或避免重新渲染。  

### 参数  

- include: 字符串或正则表达式。只有名称匹配的组件会被缓存。  
- exclude: 字符串或正则表达式。任何名称匹配的组件都不会被缓存。  
- max: 数字。最多可以缓存多少组件实例。  

### 用法  

``<keep-alive>`` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 ``<transition>`` 相似，``<keep-alive>`` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。  

当组件在 ``<keep-alive>`` 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。而被缓存的组件，只会在第一次被创建的时候，调用created，所以获取进入页面获取数据的调用，放在activated中。

    <keep-alive include="a">
        // 只有组件名为a的，才会被缓存
        <component></components>
    </keep-alive>  
  
    <keep-alive exclude="b">
      // 组件名为b的，不会被缓存
      <component></component>
    </keep-alive>
  

## 结合``<route-view>``  

``<router-view>`` 组件是一个 functional 组件，渲染路径匹配到的视图组件。<router-view> 渲染的组件还可以内嵌自己的 <router-view>，根据嵌套路径，渲染嵌套组件。    

如果想让某些视图进行缓存，某些视图不进行缓存，可以对routes中的meta属性进行修改。如：  
    
    {
        "path": "/a",
        component: r => require(["./a.vue"], r),
        meta: {
            keepAlive: false
        }
    }

    <keep v-if="$route.meta.keepAlive">
        <rotute-view></route-view>
    </keep-alive>
    
    <route-view v-else>
    </route-view>
    

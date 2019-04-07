类型： Object | Function  

限制： 组件中，data必须设置为Function  

实例创建后，可以通过vm.$data进行访问。Vue实例也代理了data上的所有属性，所以，vm.a 等价于 vm.$data.a。  

在组件中，data必须是一个返回初始数据对象的函数。因为组件可能被实例化多次，如果data是一个对象，那么各个组件将会共享这个data。通过提供 data 函数，每次创建一个新实例后，我们能够调用 data 函数，从而返回初始数据的一个全新副本数据对象。  

和原型链一样：  

  function Component() {
  }
  
  Component.prototype.data = {
    a: 1
  }; 
  
创建组件时，如果data是一个字面量的对象，所有组件将会公用一个data，造成污染。

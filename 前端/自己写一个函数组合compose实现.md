## 函数组合  

所谓函数组合，就是将函数引用作为参数，通过迭代调用函数的过程  

比如：  

```javascript
let add = (a) => a + 1
let multi = (a) => a * 2  
let r = compose(add, multi)(1) 
//====> r = 3 
// 过程为：
// add(multi(1))
```  

## 实现  

```javascript
function compose() {
    let methods = [].slice.call(arguments)
    return function () {
        let len = methods.length
        let i = len - 1
        let result = methods[i].apply(this, arguments)
        while (i > 0) {
            result = methods[--i].call(this, result)
        }
        return result
    }
}
```

es6实现  

```javascript
let compose = (...methods) => x => methods.reduceRight((y, f) => f(y), x)  
```

## 管道  

在shell中，管道操作是非常实用的方法, 如：

```shell
# 查看nginx进程树
> pstree -p | grep nginx    
```

其实管道的实现就是compose的反向操作  

```javascript
let pipe = (...methods) => x => methods.reduce((y, f) => f(y), x)
```
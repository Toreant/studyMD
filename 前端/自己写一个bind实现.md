## 需要实现的功能  

> bind()方法创建一个新的函数，在调用时设置this关键字为提供的值。并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项。

我们先看以下原生bind的例子  

```javascript
var obj = {
    val: 3
}

function fn(name) {
    console.log(this.val)
    console.log(name)
}

let newFn = fn.bind(obj, "name")
newFn() 
// or:
// newFn = fn.bind(obj)
// newFn("name")
// 上面两个都是输出以下内容
// console:
// 3
// name
```

## 实现this  

看完上面原生的例子，我们先实现this的赋值  

```javascript
Function.prototype.bind2 = function (ctx) {
    if (typeof this !== 'function') {
        throw "bind2 should call by function"
    }

    return function () {
        return this.apply(ctx)
    }
}  
```

## 传入参数  

```javascript
Function.prototype.bind2 = function (ctx) {
    if (typeof this !== 'function') {
        throw "bind2 should call by function"
    }

    let args = Array.prototype.slice.call(arguments, 1)

    return function () {
        return this.apply(ctx, [...args, ...arguments])
    }
}  
```

## 问题  

bind函数有一个重要的特点：
> 绑定函数也可以使用new运算符构造，它会表现为目标函数已经被构建完毕了似的。提供的this值会被忽略，但前置参数仍会提供给模拟函数。
> [参考mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Description)  

如下面的例子：

```javascript
function person(name) {
    this.name = name
}

let obj = {
    val: 2
}

let Men = person.bind2(obj)
let men = new Men("name")
console.log(men.name) // undefined
```

改进： 

```javascript
Function.prototype.bind2 = function (ctx) {
    if (typeof this !== "function") {
        throw "bind2 should call by function"
    }

    let self = this
    let args = Array.prototype.slice.call(arguments, 1)

    function fNOP () {}

    function bindFn () {
        if (this instanceof bindFn) {
            // 进行new操作
            return self.apply(this, [...args, ...arguments])
        } else {
            return self.apply(ctx, [...args, ...arguments])
        }
    }
    if (this.prototype) {
        // Function.prototype doesn't have a prototype property
        fNOP.prototype = this.prototype // 继承调用函数的prototype（如果有）
    }   
    
    // 下行的代码使bindFn.prototype是fNOP的实例,因此
    // 返回的bindFn若作为new的构造函数,new生成的新对象作为this传入bindFn,新对象的__proto__就是fNOP的实例
    bindFn.prototype = new fNOP() 
    return bindFn
}
```

## 注意  

- 这部分实现创建的函数有 prototype 属性。（正确的绑定函数没有）  
- 这部分实现创建的绑定函数所有的 length 属性并不是同ECMA-262标准一致的：它创建的函数的 length 是0，而在实际的实现中根据目标函数的 length 和预先指定的参数个数可能会返回非零的 length。
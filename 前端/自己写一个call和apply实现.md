call和apply很相似  

相同点:
1. 传入context，修改函数的this
2. 立即执行

不同点：
1. call调用的参数是call(context,arg1, arg2, arg3, ...), 而applly调用的参数是apply(context, [arg1, arg2, arg3, ...])  

## 实现  

call

```javascript
Function.prototype.call2 = function (ctx) {
    if (typeof this !== 'function') {
        throw new TypeError('call2 should call by function')
    }

    ctx = ctx || window
    let args = Array.prototype.slice.call(arguments, 1)
    ctx.fn = this
    let result = ctx.fn(...args)
    delete ctx.fn
    return result
}
```

apply  

```javascript
Function.prototype.apply2 = function (ctx) {
    if (typeof this !== 'function') {
        throw new TypeError('call2 should call by function')
    }

    ctx = ctx || window
    let args = arguments[1] || []

    ctx.fn = this
    let result = ctx.fn(...args)
    delete ctx.fn
    return result
}
```
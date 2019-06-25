## 函数柯里化  

函数柯里化：它用于创建已经设置好了一个或多个参数的函数。函数柯里化的基本方法和函数绑定是一样的：使用一个闭包或返回一个函数。两者的区别在于，当函数被调用时，返回的函数还需要设置一些传入的参数。  

## 通用的基本柯里化  

    function curry(fn) {
        let args = [].slice.call(arguments, 1)
        return function () {
            let finalArgs = args.concat([].slice.call(arguments))
            return fn.apply(this, finalArgs)
        }
    }
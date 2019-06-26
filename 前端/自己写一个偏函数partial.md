## 偏函数  

在计算机科学中，局部应用是指固定一个函数的一些参数，然后产生另一个更小元的函数。

什么是元？元是指函数参数的个数，比如一个带有两个参数的函数被称为二元函数。

例子：  

    let add = function (a, b) {
        return a + b
    }

    let addFn = partial(add, 1)
    addFn(2)
    //====> 1 + 2 = 3  


## 实现  

    function partial(fn) {
        let args = [].slice.call(arguments, 1) // 取出partial函数中，除了fn后面的参数
        return function () {
            let localArgs = [].slice.call(arguments)
            return fn.apply(this, args.concat(localArgs))
        }
    }  

## 实现占位符  

    let _ = Symbol('_')

    function partial(fn) {
        let args = [].slice.call(arguments, 1)
        return function () {
            let position = 0
            for (let i = 0; i < args.length; i++) {
                args[i] = args[i] === _
                    ? arguments[position++]
                    : args[i]
            }

            // 把arguments剩下的参数添加到args
            while (position < arguments.length) {
                args.push(arguments[position++])
            }

            return fn.apply(this, args)
        }
    }  

## 偏函数和函数柯里化的区别  

相同点  
1. 都是返回一个函数  

不同点  
1. 柯里化是将一个多参的函数，转化成多个一个参数的函数，也就是将n元函数转化成n个一元函数
2. 偏函数是将一个多参的函数，转化成固定参数的函数，也就是将n元函数转化成n-x元函数
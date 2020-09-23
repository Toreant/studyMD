## 函数柯里化  

函数柯里化：它用于创建已经设置好了一个或多个参数的函数。函数柯里化的基本方法和函数绑定是一样的：使用一个闭包或返回一个函数。两者的区别在于，当函数被调用时，返回的函数还需要设置一些传入的参数。  

## 通用的基本柯里化  

```javascript
var curry = function (fn) {
    var argsLen = fn.length // fn函数需要的参数个数
    var args = [].slice.call(arguments, 1) // 额外提供的参数

    return function () {
        var finalArgs = args.concat([].slice.call(arguments))
        if (finalArgs.length === argsLen) {
          return fn.apply(this, finalArgs)
        } else {
          return curry.apply(this, [fn].concat(finalArgs))
        }
    }
}
```

## 可以占位的柯里化   

```javascript
let _ = Symbol('_') // 占位符

function curry(fn, args, holes) {

    let length = fn.length
    args = args || []
    holes = holes || []

    return function () {

        let _args = args.slice(0)
        let _holes = holes.slice(0)
        let _argsLen = _args.length
        let _holeLen = holes.length
        let i, index = 0

        for (i = 0; i < arguments.length; i++) {
            let arg = arguments[i]
            if (arg === _ && _holeLen) {
                // 处理类似 fn(1, _, _, 4)(_, 3) 这种情况，index 需要指向 holes 正确的下标
                index += 1
                if (index > _holeLen) {
                    _args.push(arg)
                    _holes.push(_argsLen - 1 + index - _holeLen)
                }
                
            } else if (arg === _) {
                // 处理类似 fn(1)(_) 这种情况
                _args.push(arg)
                _holes.push(_argsLen + i)
            } else if (_holeLen) {
                if (index >= _holeLen) {
                    // fn(_, 2)(_, 3)中的3
                    _args.push(arg)
                } else {
                    // fn(_, 2)(1) 用参数 1 替换占位符
                    _args.splice(_holes[index], 1, arg)
                    _holes.splice(index, 1)
                }
            } else {
                _args.push(arg)
            }

        }
        //console.log(_args, _holes)

        if (_holes.length || _args.length < length) {
            // 代表还没有传入所有的参数，返回一个curry化后的函数，
            return curry.call(this, fn, _args, _holes)
        } else {
            return fn.apply(this, _args)
        }
    }
}

let add = function (a, b) {
    return a + b
}

let addFn = curry(add)
let add1 = addFn(1, _)(2)
let add2 = addFn(1, 2)
let add3 = addFn(_, 2)(1)
let add4 = addFn(_ , _)(1, 2)
```

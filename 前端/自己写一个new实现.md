## new操作符  

在javascript中，new操作符究竟实现了什么呢？  

    function Men(name) {
        this.name = name
    }

    Men.prototype.getMessage = function () {
        return `My name is ${this.name}`
    }

    let man = new Men("name")  
    let r = man.getMessage()
    console.log(r) // My name is name

大概看出，new操作符里面执行了以下内容：

1. 创建新的对象
2. 设置原型链，__proto__指向prototype对象  
3. this指向新的对象 

那么，通过上面内容，可以写出一个简易的new操作符操作。由于new是关键字，所以，使用下面代码来代替  

    function newInstance(fn, arg1, arg2, ...) {}    

## 代码实现  

    function newInstance() {
        let Constructor = [].shift.call(arguments) // 取出第一个参数，并修改了arguments，使得arguments是剩下的参数数组
        let obj = new Object()
        obj.__proto__ = Contructor.prototype
        Constructor.apply(obj, arguments)
        return obj
    }


## 改进  

上面代码没有问题，但是，需要进一步考虑函数的返回值  

    function Men(name, age) {
        this.name = name
        return {
            age: age,
            sex: 'male'
        }
    }
    // Men函数返回一个对象，
    let man = new Men("jack", 12)
    console.log(man.name) // undefined
    console.log(man.age) // 12
    console.log(man.sex) // male
    console.log(man) // {"age": 12, "sex": male}  

看出来，如果函数返回的是一个引用值，需要直接返回这个值，this不再指向新建的对象  

    function newInstance() {
        let Constructor = [].shift.call(arguments)
        let obj = new Object()
        obj.__proto__ = Contructor.prototype
        let r = Constructor.apply(obj, arguments)
        return typeof r === 'object' ? r : obj
    }

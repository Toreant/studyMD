## Promise 

- promise有三个状态，pending、fulfilled、rejected。同时，fulfiled和rejected只能从pending状态改变，而且，fulfilled、rejected状态不能改变成其他状态。  

- promise必须有then方法，then方法返回一个Promise对象。

- then方法接受两个函数, onFulfilled和onRejected。

	- 当promise状态为fulfilled时，执行onFulfilled函数，并只执行一次。

	- 当promise状态为rejected时，执行onRejected函数，并只执行一次。  


以下是实现思路：  

	function Promise(fn) {
		let state = 'pending';
		let value = null;
		let deffered = null;

		this.then = function (onFulfilled, onRejected) {
			return new Promise(function (resolve, reject) {
				handle({
					onFulfilled: onFulfilled,
					onRejected: onRejected,
					resolve: resolve,
					reject: reject
				});
			});
		};

		function resolve(newValue) {}

		function reject(newValue) {}

		function handle(handler) {
			setImmediate(function () {
				// do somthing
			});
		}

		fn(resolve, reject);
	}


上面是一个Promise对象的大概构造过程，可以看到，构造函数里面执行的是同步操作，但是，then是一个异步操作的过程，这也是Promise规定。 为什么then里面是一个异步操作呢？是因为，一般promise会进行很多个then连接，如果then里面是一个同步过程，promise执行可能需要很长时间，造成后面的程序等待。所以，then里面可以用一个macro-task（setTimeout、setImmediate）或micro-task（process.nextTick）等，把then的程序放到事件队列的后面，避免主程序卡顿。

重要的一点，resolve函数需要回传数据，而reject函数第一个参数必须是错误的信息。  


    function resolve(newValue) {
        // resolve函数可能接收一个promise，所以这里判断一下
        if (newValue && typeof newValue.then === 'function') {
            newValue.then(resolve, reject);
            return;
        }

        state = 'fulfilled';
        value = newValue;

        if (deffered) {
            deffered(value);
        }
    }


### Promise中函数  

#### Promise.all  

接收promise数组，当所有的promise数组成功执行完毕后，返回成功结果数组的promise。如果某个promise失败，则返回失败信息，并且终止剩下的操作。

#### Promise.once  

接收promise数组，如果有一个成功执行完毕，则返回当前成功结果，并终止剩下操作。


### 参考链接  

[深入理解Promise](http://coderlt.coding.me/2016/12/04/promise-in-depth-an-introduction-2/)  
[promises-in-wicked-detail](https://www.mattgreer.org/articles/promises-in-wicked-detail/)  
[promise/A+规范](https://promisesaplus.com/)
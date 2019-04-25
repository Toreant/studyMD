## 数组快速去重  

es6写法  
```javascript
[...new Set(array)]

// or
Array.from(new Set(arr))
```    

es5写法   
```javascript
array.filter((item, index) => index === array.indexOf(item))
```  

## 快速取整  

```javascript
var a = ~~1.983    // 1
var b = 1.987 | 0  // 1  
var c = 1.987 >> 0 // 1
```  
以上代码都是向下取整  

## 快速对数字金钱格式化  

```javascript
var a = (num + '').replace(/\B(?=(\d{3})+(?!\d))/g, ',');
```  

其实javascript中也是有官方的方法  

```javascript
(num).toLocaleString('en-US')
```

非正则方法  

```javascript
function formatCash(str) {
      return str.split('').reverse().reduce((prev, next, index) => {
            return ((index % 3) ? next : (next + ',')) + prev
       })
}
```  

## 取出数组中的最大、最小值  

```javascript
var number = [1,2,3,4,5];
var max = Math.max.apply(Math, number);
var min = Math.min.apply(Math, number);
```  

## 交换两个数 

```javascript
var a = 1;
var b = 2;

a ^= b;
b ^= a;
a ^= b;
```  

当然，es6中的解构赋值可以快速解决问题  

```javascript
let a = 1;
let b = 2;
[a, b] = [b, a];
```  

## RGB to Hex  

```javascript
function toHEX(rgb){
	return ((1<<24) + (rgb.r<<16) + (rgb.g<<8) + rgb.b).toString(16).substr(1);
}
```  

## 把类数组转化为数组

```javascript
// es5版本
[].slice.call(arr);

// es6版本
Array.from(arr)
```
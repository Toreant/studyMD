### 数据处理方式  

1. react中的数据state修改，需要通过setState来触发视图的更新。showComponentUpdate判断是否需要更新视图，如果返回false，则不进行下一步的判断。  
2. vue中的数据是响应式的，就是说，如果data改变了，vue会自动触发视图的更新。里面的实现原理是，通过对data的键值进行watch操作，劫持data数据的set操作或数组的操作【push|pop|splice|shift|unshift|sort|reverse】

### 代码的编写  

1. react把所有的代码写进了js里面，其中包括视图的逻辑，所以，react使用了jsx来编写视图。

2. vue则是把html\js\css可以写进一个文件里面。  

vue更加直观一点，不过react更加符合纯组件的设想。  

### 组件写法 

1. react是类的写法

2. vue则是声明式的写法


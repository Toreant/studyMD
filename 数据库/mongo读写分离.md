上一篇分享我介绍了如果搭建一个副本集的mongo环境，[传送门](https://blog.toreant.top/article/5979ba7d32ba7b494a5de10b)。但是，里面有一个问题，就是，primary承担着所有的写操作，同时，primary还需要负责读操作，这样，对一个服务器来说，可能压力会有点大。所以，我们可以充分利用其他副节点，来分担primary的压力。  

## 读写偏好  

由于mongo规定，写操作只能在primary上进行，所以，我们可以把读操作放在secondaries上。mongo中有一个读写偏好的选项，  

### primary  
primary是mongo默认的设置，每次读操作都从primary中进行。  

### primaryPreferred  
primary作为第一选择，当primary不可读时，从secondaries中选取。  

### secondary  
读操作只从副本集的secondary成员上读数据。如果没有secondary可用，则产生一个error或者抛出一个异常。  

### secondaryPreferred  
读操作首选副本集的secondary成员上读数据，如果没有secondary成员，则从primary上读数据。  

### nearest  
驱动从最近的集合成员中读数据时一个成员选择的过程。该模式不关注成员的类型，不管是primary还是secondary成员。  

## nodejs中的使用  
以mongoose包为例，设置读写偏好是在定义Schema时确定的， 
```javascript  
var schema = new Schema({..}, { read: 'primary' });            // also aliased as 'p'  
var schema = new Schema({..}, { read: 'primaryPreferred' });   // aliased as 'pp'  
var schema = new Schema({..}, { read: 'secondary' });          // aliased as 's'  
var schema = new Schema({..}, { read: 'secondaryPreferred' }); // aliased as 'sp'  
var schema = new Schema({..}, { read: 'nearest' });            // aliased as 'n'  
```
## 读写分离思考  
1. 由于副本集是一个primary进行写操作、并把写命令发送到副节点上，这里存在着网络延迟的问题，可能副节点上的数据并不是最新的数据。所以，如果对实时性需求比较大的数据，最好还是从primary上进行读取。  
2. 如果考虑到网络问题，最好还是从网络延迟最小的节点进行读操作（nearest）。

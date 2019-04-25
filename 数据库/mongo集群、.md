mongo是一个优秀的nosql数据库，在海量数据存储中，mongo起着很大作用。但是，mongo也是一个相当耗内存的一个服务，如果设置不当，就会随时发生宕机的可能（服务器上确实发生过多次宕机的情况）。为了避免mongo宕机导致应用服务出现访问不了的情况，mongo提供了集群的功能。  

在这里，我们使用了replica-set模式，是一种副本集模式。

**Replica Set**
副本集模式，简单的说，就是有多个mongo服务器，每个服务器都备份了相同的数据，但Primary挂掉后，会选出一个Secondaries作为新的master。这样，mongo服务就不会出现无法提供服务的情况。
![enter image description here](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg "enter image title here") 

> **Primary**:  
>   The primary receives all write operations.   
  

> **Secondaries**:  
> Secondaries replicate operations from the primary to maintain an identical data set. Secondaries may have additional configurations for special usage profiles. For example, secondaries may be non-voting or priority 0.  

### 第一步  

首先建立三个主从服务器（mongo2, mongo3, mongo4），一个监督服务器(mongo5)。

![enter image description here](../upload/bf35c1ae3ae8168ba898243b5d7f06d7.jpg "enter image title here")  
 
在每个目录下添加config文件，

#### primary  
![主服务器配置](https://blog.toreant.top/img/upload/2768efdecc90f3453549b78ff0367fa3.jpg)  

#### secondaries1
![从服务器1](../upload/717a5fe5ea3f3ce818c32ac2eb69bbbe.jpg)  

#### secondaries2  
![从服务器2](../upload/a23803b79c1c466b4dc27277a2000585.jpg)  

#### 监督服务器  
![监督服务器](../upload/09305ad7917bfcdf0d453068f36e1caf.jpg)  

### 启动每个服务器
![启动](../upload/b3f20df09d869c4f941857884c735065.jpg) 

### 进入主服务器管理  
![admin](../upload/3d24c16cd81d17300d8f4e3a4c877553.jpg)  

### 启用replica Set  
![replica set](../upload/c911584ad8e14585ca15b6f5f7e20462.jpg)  

### 设置监督服务器  
![set](../upload/fd48cd314b81cfd591adcb4d0a55addd.jpg)  

### 查看状态  
![status](../upload/6308cc06b5d4a9fdfc5b474f6b463544.jpg) 
![status2](../upload/1db8db387dabd93cad0abad6b14eba3d.jpg)  

可以看到，端口号为27020的mongo服务器已经作为主服务器了。

同时，主服务器接收所有的write请求，并向secondaries发送消息，进行数据同步。
![replication](../upload/c8f9ac9cde234c35f9f53bf5ed0434e2.jpg) 

这样，数据就存储在不同的mongo服务器中，当primary宕机发生，也不用害怕数据丢失了。

primary挂掉后，监督服务器会在剩下的secondaries中投票胜出的服务器作为新的primary，而原来的primary会在重新启动后，添加进secondaries中。  

### nodejs中的使用   

我使用了mongoose包，如果需要使用replica-set模式，可以在配置中添加多个链接，代码如下：  

```javascript
let serverConfig = {
    replset: {
        pollSize: 3
    }
};

mongoose.Promise = global.Promise;
let mongo = 'mongodb://localhost:27021,localhost:27020,localhost:27022/max';
let conn1 = mongoose.createConnection(mongo, serverConfig, (err) => {
    if (err) {
        console.log(('数据库1链接错误:' + err.message));
        process.exit(1); //离开数据库
    } else {
        console.log('数据库1链接成功');
    }
});
```  

mongoose中有一句话：  
> This connection object is then used to create and retrieve models. Models are always scoped to a single connection.    


意思就是说，“这个链接会用来创建和检索model，同时model总是使用一个链接”。也就是说，mongoose会在一开始进行连接时选择合适的mongo数据库，后面的数据操作都是使用当前这个数据库，而这个数据库就是primary。  

由于当前primary是端口为27020，我把27020删除  

```javascript
let serverConfig = {
    replset: {
        pollSize: 3
    }
};

mongoose.Promise = global.Promise;
let mongo = 'mongodb://localhost:27021,localhost:27022/max';
let conn1 = mongoose.createConnection(mongo, serverConfig, (err) => {
    if (err) {
        console.log(('数据库1链接错误:' + err.message));
        process.exit(1); //离开数据库
    } else {
        console.log('数据库1链接成功');
    }
});
```   

也还是可以正常工作的。  

不过如果我删除只剩一个时，就会出现错误了  
```javascript
let serverConfig = {
    replset: {
        pollSize: 3
    }
};

mongoose.Promise = global.Promise;
let mongo = 'mongodb://localhost:27021/max';
let conn1 = mongoose.createConnection(mongo, serverConfig, (err) => {
    if (err) {
        console.log(('数据库1链接错误:' + err.message));
        process.exit(1); //离开数据库
    } else {
        console.log('数据库1链接成功');
    }
});
```   

它会提示我说当前连接不是master。  

顺便说一下，replica-set模式并不是为了进行读写分离、减轻服务器压力的举措，它是为了数据冗余、备份。由于replica-set是通过复制操作进行数据备份，可能会发生网络原因，副节点没有更新到最新的数据，如果从副节点进行读操作，会读取到旧的数据。

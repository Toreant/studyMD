## webpack热更新（Hot-Module Replacement）  

### 配置实现  

- webpack添加热更新插件  


        // webpack.conf.js
        plugins: [
            ...,
            new webpack.HotModuleReplacementPlugin({...})
        ]  


- 使用webpack-hot-middleware 

        // dev-server.js
        const app = express();
        const compiler = webpack(webpackConfig);

        const devMiddleware = require('webpack-dev-middleware')(compiler, {
            publicPath: webpackConfig.output.publicPath,
            quiet: true
        });

        const hotMiddleware = require('webpack-hot-middleware')(compiler, {
            log: false,
            heartbeat: 2000
        });

        app.use(hotMiddleware);
        app.use(devMiddleware);  

### 热更新过程    

webpack-hot-middleware使用了``Server-send Events``进行通信。由于sse长时间没有消息的话，连接会断开，所以，服务器会每隔一段时间发送一个心跳包，以保持链接。    

sse传过来的message，里面的内容是：  

    {
        “name": "",
        "action": "sync",
        "hash": "",
        "warnings": [],
        "errors": [],
        "modules": []
    }  

修改代码后，发现，sse发送了一个message:  

    {
        "action": "building"
    }  

接着，sse有发送了一个message:  

    {
        "name":"",
        "action": "built",
        "hash": hash
    } 
     
同时，浏览器发送ajax请求，获取hot-update.json，得到：  

    {
        "id": 20, // 需要更新的模块id
        "h": hash
    }  

接着，浏览器继续请求20-xxx.hot-update.js，里面的内容：  

    // 调用webpackHotUpdate方法
    webpackHotUpdate(20, {
        eval('。。。更新模块的代码。。。')
    })  


### 热更新实现分析  

node与浏览器通信 

node通过中间件：[webpack-hot-middleware/middleware.js](https://github.com/webpack-contrib/webpack-hot-middleware/blob/master/middleware.js)  

创建createEventStream:  

    function createEventString(heartbeat) {
        return {
            handler: function(req, res) {
                var headers = {
                    'Access-Control-Allow-Origin': '*',
                    'Content-Type': 'text/event-stream;charset=utf-8',
                    'Cache-Control': 'no-cache, no-transform',
                    // While behind nginx, event stream should not be buffered:
                    // http://nginx.org/docs/http/ngx_http_proxy_module.html#proxy_buffering
                    'X-Accel-Buffering': 'no', 
                };

                var isHttp1 = !(parseInt(req.httpVersion) >= 2);
                if (isHttp1) {
                    req.socket.setKeepAlive(true);
                    Object.assign(headers, {
                    Connection: 'keep-alive',
                    });
                }

                res.writeHead(200, headers);
                res.write('\n');
                var id = clientId++;
                clients[id] = res;
                req.on('close', function() {
                    if (!res.finished) res.end();
                    delete clients[id];
                });
            },

            publish: function(payload) {
                everyClient(function(client) {
                    client.write('data: ' + JSON.stringify(payload) + '\n\n');
                });
            },
        }
    }

发送消息给客户端：  

- sync ：文件修复热更新或者报错会发送该消息
- building ：编译中
- built ：编译完成  

客户端实现：  

    var source = new window.EventSource('(http://127.0.0.1:9000/__webpack_hmr)');
    source.onopen = handleOnline; // 建立链接
    source.onerror = handleDisconnect;
    source.onmessage = handleMessage; // 接收服务端消息，然后进行相应处理  

客户端热更新：  

- 模块初始化：  

        // 在加载模块的方法中，为每个模块都添加了热更新的逻辑 
        function __webpack_require__(moduleId) {
            var module = installedModules[moduleId] = {
                i: moduleId,
                l: false,
                exports: {},
                hot: hotCreateModule(moduleId), // 前端通过 ajax 获取热更新文件内容
                parents:xxxx,
                children: []
            };
            return module.exports;
        }

- hotCreateModule：

        function hotCreateModule(moduleId) {
            accept: function() {},
            check: hotCheck,
            apply: hotApply
        }  

- hotCheck: 获取更新文件的内容

        function hotCheck(apply) {
            for(var chunkId in installedChunks)
 			// eslint-disable-next-line no-lone-blocks
 			{
 				/*globals chunkId */
 				hotEnsureUpdateChunk(chunkId);
 			}
            if (
 				hotStatus === "prepare" &&
 				hotChunksLoading === 0 &&
 				hotWaitingFiles === 0
 			) {
 				hotUpdateDownloaded();
 			}
        }  

- hotUpdateDownloaded: 判断是否下载更新  

        function hotEnsureUpdateChunk(chunkId) {
            if (!hotAvailableFilesMap[chunkId]) {
                hotWaitingFilesMap[chunkId] = true;
            } else {
                hotRequestedFilesMap[chunkId] = true;
                hotWaitingFiles++;
                hotDownloadUpdateChunk(chunkId);
            }
        } 

- hotDownloadUpdateChunk： 下载更新模块文件

        function hotDownloadUpdateChunk(chunkId) {
            var head = document.getElementsByTagName("head")[0];
            var script = document.createElement("script");
            script.charset = "utf-8";
            script.src = __webpack_require__.p + "" + chunkId + "." + hotCurrentHash + ".hot-update.js"; // 
            if (null) script.crossOrigin = null;
            head.appendChild(script);
        }  

开启热更新模块后，每个Vue组件都会有这么一段热更新代码： 

    /* hot reload */
    Component.options.__file = "app/web/page/about/about.vue"
        if (true) {(function () {
        var hotAPI = __webpack_require__(3)
        hotAPI.install(__webpack_require__(0), false)
        if (!hotAPI.compatible) return
        module.hot.accept()
        if (!module.hot.data) {
            hotAPI.createRecord("data-v-aafed0d8", Component.options)
        } else {
            hotAPI.reload("data-v-aafed0d8", Component.options)
        }
        module.hot.dispose(function (data) {
            disposed = true
        })
    })()}  

通过createRecord或reload出发视图的更新。  


### 总结：  

1. webpack编译阶段，为每个module注入热更新代码逻辑
2. 页面首次打开后，服务器与客户端通过sse进行连接，把下一次的hash传给前端
3. 客户端获取到hash，这个hash将作为下一次请求服务端 hot-update.js 和 hot-update.json的hash
4. 修改代码后，webpack触发更新逻辑，服务器发送message（{action: building}）到客户端，
5. 客户端获取到hash，成功后客户端构造hot-update.js script链接，然后插入主文档
6. hot-update.js 插入成功后，执行hotAPI 的 createRecord 和 reload方法，获取到 Vue 组件的 render方法，重新 render 组件， 继而实现 UI 无刷新更新。
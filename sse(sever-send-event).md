在nginx中，代理server-send event，需要添加以下头部：

    X-Accel-Buffering: no
    Cache-Control: no-cache
    Connection: keep-alive
    Content-Type: text/event-stream

因为nginx默认会使用缓存区，但是，server-send event需要不断输送内容，所以缓存区是会影响的。 

server-send event是不能添加而外头部的，需要添加鉴权的话，可以在url中添加鉴权信息。

nginx对server-send event的相应中，会默认添加``Transfer-Encoding: chunked``的头部。因为server-send event是没法知道服务端回传内容的大小的，所以不会对相应添加``Content-Length``头部，nginx会默认对其添加``Transfer-Encoding: chunked``来标示此相应是一个分段内容。
## XSS  

XSS, 又称CSS(cross-sitescript)，跨域脚本攻击。类似于sql注入，攻击者通过输入恶意的HTML代码，用户访问这段代码时自动执行了，从而达到攻击目的。  

### 防御  

一般来说，防止XSS攻击的方法是对用户的输入进行过滤。  

### nodejs XSS防御  

- xss.js模块  

- 前端dom操作  
    - 在前端不清楚服务器返回的内容是否安全的情况下，可以把内容转化成文本节点。  
    - 方式1：document.craeteTextNode(input)   


## CSRF  

CSRF，cross-site requst Forgery，跨站点伪造请求。CSRF通过伪装成受信任用户的请求来利用受信任的网站。  

### nodejs CSRF防御  

- csurf模块  
    - 前端发起请求，获取token，服务器建立cookie和token的对照关系。前端把token存起来。每次发送请求时，带上cookie和token，服务器进行验证。  
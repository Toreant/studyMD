# 使用Visual Studio 2017编译grpc  

## 编译grpc   

1. 下载grpc源码  
``git clone https://github.com/grpc/grpc.git``

2. 安装grpc子模块和第三方依赖  
``cd grpc && git submodules update --init``  

3. windows安装cmake, choco

4. 安装activeperl  - required by boringssl  
``choco install activeperl``  

5. 安装golang   - required by boringssl  
``choco install golang``  

6. 安装yasm   - required by boringssl  
``choco install yasm``  

7. 我的机器是64位系统，所以使用cmake编译时，需要按照64位进行编译。  
8. 新建文件夹，编译好的内容会放到这里。   
``mkdir .build_x64 && cd .build_x64``  

9. 使用cmake生成工程文件  
``cmake .. -G "Visual Studio 15 2017 X64" -DCMAKE_BUILD_TYPE=Release`` 

10. 用Visual Studio打开工程文件``.build_x64/gpr.vcxproj``，然后进行编译、生成。 

## 使用grpc  

新建项目，把grpc的相关库引入。 不过其中有些Visual Studio需要设置。  

> 生成 》 配置管理器 》 活动解决方案平台， 设置为x64     
> 属性 》 C/C++ 》 预处理器 》 预处理器定义，添加_WIN32_WINNT=0x600  

然后，添加grpc相关的头文件。   
> 属性 》 C/C++ 》 常规 》 附加包含目录， 把下面的相关目录添加进去。  
<pre>
PATH_TO_GRPC\include
PATH_TO_GRPC\third_part\protobuf\src
PATH_TO_GRPC\third_part\address_sorting\include
PATH_TO_GRPC\third_part\cares\cares
</pre>  

添加lib文件目录  
> 属性 》 链接器 》 常规 》 附加库目录  
<pre>
PATH_TO_BUILD\Debug
PATH_TO_BUILD\third_party\protobuf\Debug
PATH_TO_BUILD\third_party\boringssl\crypto\Debug
PATH_TO_BUILD\third_party\boringssl\ssl\Debug
PATH_TO_BUILD\third_party\zlib\Debug
PATH_TO_BUILD\third_party\cares\cares\lib\Debug
</pre> 

添加lib  
> 属性 》 链接器 》 输入 》 附加依赖项  
<pre>
grpc++.lib
gpr.lib
libprotobufd.lib
ssl.lib
crypto.lib
zlibstaticd.lib
address_sorting.lib
Ws2_32.lib
cares.lib
</pre>  

## 编译proto 

grpc编译好，在.build_x64\Debug下，有两个程序``grpc_cpp_plugin.exe``和``protoc.exe``，这两个是用来编译proto文件，生成头文件和原文元的。 把这两个程序复制到项目中去，然后新建一个.proto文件，例如： line.proto. 对这个proto文件进行设置：  
> 常规 》 项类型， 设置为自定义生成工具，点击应用。  
> 自定义生成工具 》 工具，添加命令行  
> ``protoc --cpp_out=. line.proto``  
> ``protoc --grpc_out=. --plugin=protoc-gen-grpc=grpc_cpp_plugin.exe line.proto``    

编译生成了，``line.pb.h``, ``line.pb.cc``, ``line.grpc.pb.h``, ``line.grpc.pb.cc``  

## 构建grpc应用  

1. 新建空白项目，按照使用grpc的方法，添加grpc相关设置。
2. 添加已有项目，把上一步proto生成的头文件和源文件添加进去。    

## 构建基于grpc的dll  

1. 新建dll项目
2. 设置C/C++ 》 预编译头，不适用预编译头  

这里通过dll导出一个class，首先添加一个类，里面是一些纯虚函数, 
RpcProxy.h  
<pre>
#pragma once
#ifdef RPCPROXY_EXPORTS
#define RPCPROXY_API __declspec(dllexport)
#else
#define RPCPROXY_API __declspec(dllimport)
#endif

#include <string>

class RpcProxy
{
public:
	virtual char* recieveData(const char* line, const char* id) = 0;
	virtual std::string recieveData() = 0;
	virtual void Release() = 0;
};

extern "C" RPCPROXY_API RpcProxy* _stdcall CreateExportObj(); // 这里通过导出一个函数，生成一个RpcProxy类
extern "C" RPCPROXY_API void _stdcall DestroyExportObj(RpcProxy* rpcProxy); // 这里导出一个函数，用来销毁RpcProxy类
</pre>

用一个导出类来继承RpcProxy类，里面包含着虚函数， ExportTpl.h  
<pre>
#pragma once
#include <string>
#include "rpcProxy.h"
#include "client.h"

class ExportTpl : public RpcProxy
{
public:
	virtual char* recieveData(const char* line, const char* id);
	virtual std::string recieveData();
	virtual void Release();
	ExportTpl();
	~ExportTpl();
private:
	Client* client; // grpc_client
};
</pre>

RpcProxy.cpp，定义dll应用程序的导出函数  
<pre>
// rpcProxy.cpp : 定义 DLL 应用程序的导出函数。
//
#include "stdafx.h"
#include "rpcProxy.h"
#include "ExportTpl.h"

#ifdef _MANAGED
#pragma managed(push, off)
#endif

#ifdef _MANAGED
#pragma managed(pop)
#endif

RPCPROXY_API RpcProxy* APIENTRY CreateExportObj()
{
	return new ExportTpl;
}

RPCPROXY_API void APIENTRY DestroyExportObj(RpcProxy* rpcProxy)
{
	rpcProxy->Release();
}
</pre>

ExportTpl.cpp, 定义RpcProxy类的函数  
<pre>
// ExportTpl.cpp
ExportTpl::ExportTpl()
{/*构造函数*/}

char* ExportTpl::recieveData(const char* line, const char* id)
{/*接受函数*/}

......
</pre>   

## node调用dll  

1. 在binging.gyp中，添加lib的
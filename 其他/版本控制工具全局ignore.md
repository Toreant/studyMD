## git  

在用户的根目录中新建.gitignore

```shell
git config --global core.excludesfile ~/.gitignore_global
```  

然后配置根目录中的.gitconfig

```
[core]
	excludesfile = C:/Users/yuhuajie/.gitignore
``` 


## svn  

修改用户配置中的config，把下面的注释掉  

```
global-ignores = *.o *.lo *.la *.al .libs *.so *.so.[0-9]* *.a *.pyc *.pyo __pycache__
```

或则直接在小乌龟设置

所谓的只clone某个文件，原理就是在clone的时候，过滤其他文件的过程。这里记录下具体的步骤：


```shell
## 1. 创建仓库
git init test && cd test

## 2. 设置允许克隆子目录
git config core.sparsecheckout true

## 3. 设置需要克隆的目录
echo "需要克隆的目录或文件路径" >> .git/info/sparse-checkout

## 4. 设置仓库的git地址
git remote add origin 仓库地址

## 5. clone
git pull origin master
```

# git学习
## 第一次修改：建立本文档
```bash
git add .
git commit -m "first commit"
git push -u origin main
```
## 第二次修改：ssh
增加了ssh密钥，不知道是否有用
## 第三次修改：添加lab0子模块
```bash
git submodule add git@github.com:N2Sys-EDU/lab0-introduction-SJZHZ.git
```
## 第四次修改：添加lab1子模块
```bash
git submodule add git@github.com:N2Sys-EDU/lab1-myftp-SJZHZ.git
```
## 第五次修改：加载lab1的test_local子模块
```zsh
git submodule update --init
git submodule update --remote
```
## 第六次修改：完成lab1（push自动测试），增加了data文件夹用于存放lab数据
```zsh
git add .
git commit -am "Update"
git push
```
## 第七次修改：加载lab2子模块
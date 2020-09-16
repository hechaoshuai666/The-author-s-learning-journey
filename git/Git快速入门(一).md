## Git快速入门

### 1. 什么是Git

+ 用于本地版本控制
+ 分布式版本控制

### 2. 为什么要做版本控制

要保留之前所有的版本,方便回滚和修改

### 3. Git的安装

详情 : https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git

### 4. 基本的命令使用

想要让Git对一个目录进行版本控制,需要以下步骤

+ 进入要管理的文件夹

+ 执行初始化命令

  ```bash
  git init # 会默认生成一个.git的隐藏文件,用于管理当前文件目录
  
  git clone remote_url # 该命令用于已经在远程仓库创建目录,克隆到本地
  ```

+ 管理目录下的文件状态

  ```bash
  git status 
  
  注:新增或修改过的文件颜色都为绿色
  ```

+ 管理指定文件 (红变绿)

  ```bash
  git add . # 将当前目录所有未追踪的文件添加管理
  git add file # 单个文件
  ```

+ 生成版本

  ```bash
  git commit -m '描述信息'  
  ```

你的电脑会报错?

如果是首次提交commit,在没有配置个人信息的时候,会出现错误(Git 不喜欢不愿透漏姓名的人)

+ 个人信息配置:用户名,邮箱(全局一次即可)

```bash
git config --global user.name "你的姓名"
git config --global user.email '你的邮箱'
# 上述命令为全局配置,取消--global为局部配置
```

+ 查看版本记录

  ```bash
  git log
  ```

+ 回滚到之前的版本

  ```bash
  git log
  git reset --hard 版本号
  ```

+ 回滚到之后的版本

  ```bash
  git reflog # git log 只能看当前位置和之前的版本信息,reflog可以看所有
  git reset --hard 版本号
  ```

### 5. Git的三大区域

![img](https://images2017.cnblogs.com/blog/425762/201708/425762-20170811110830683-181174888.png)

## 总结

```bash
git init 初始化仓库
git config (--global) user.xxx 'xxx' 配置个人信息
git add 添加文件到暂存区
git commit -m '信息' 添加文件到仓库区
git log 查看日志
git status 查看文件状态
git reflog 查看所有的日志信息
git reset --hard 版本号 版本回退
```


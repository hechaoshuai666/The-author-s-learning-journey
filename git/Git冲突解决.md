# Git冲突解决

假设我们此时正在更新版本,有一个dev分支,还有两个不同功能的开发分支

这里我们尚且叫它们f_login,f_register

由gitflow的工作流可知,当我们开发完自己的功能之后,自然要通过pull request把自己的

分支合并到dev上

如果f_login分支和f_register分支在同一个文件里产生了冲突,f_login先提交,dev接受合并

此时f_register再提交,自然就会产生冲突

两种解决方案

+ 方式一

  1.获取最新代码

  ```git
  git fetch origin dev # 该操作并不会直接合并,而是将代码拉到仓库区
  ```

  2.对比代码

  ```git
  git diff origin/dev
  
  # 也可借助工具来查看不同的地方
  ```

  3.修改冲突后提交代码

  4.发送合并请求

+ 方式二

  1.拉取并合并最新代码

  ```git
  git pull origin dev
  ```

  2.查看冲突代码

  ```git
  git status
  ```

  3.修改冲突代码后提交并推送代码

  4.发起合并请求
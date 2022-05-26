## Git 开发流程

### 简述

使用Git作为代码版本资源管理，因其灵活性和复杂性，需根据团队实际的开发情况。制定一套可实施的开发流程，当前主流的流程有如:

* Git flow
* Github flow
* Gitlab flow

等等，这些流程并非是银弹、也非强制。[关于这几种的比较见：<http://www.ruanyifeng.com/blog/2015/12/git-workflow.html>]

本方案，决定采用 Gitlab flow 模型，其主要考虑如下几点

* 既可以支持按版本发布
* 又便于持续集成
* 理解简单

### Gitlab flow 简介

<https://docs.gitlab.com/ee/topics/gitlab_flow.html>

<https://zhuanlan.zhihu.com/p/38774185>

是 Git flow 与 Github flow 的综合。它吸取了两者的优点，既有适应不同开发环境的弹性，又有单一主分支的简单和便利。它是 Gitlab.com 推荐的做法。

Gitlab flow 的最大原则叫做"上游优先"（upsteam first），即只存在一个主分支master，它是所有其他分支的"上游"。只有上游分支采纳的代码变化，才能应用到其他分支。

### 工具

推荐  <https://www.sourcetreeapp.com/> 、<https://www.gitkraken.com/>(这个谁来试试，据说完美得狠 ，但私库收费)

不推荐 tortoiseGit

### 学习

可在 www.github.com 建立自己的仓库，自己学习

完整的git文档在： <https://git-scm.com/book/zh/v2>  (这个是全面的、复杂的)

命令行帮助： git help 命令 。 比如: git help log

网上某些资料是比较过时的，比如老版本的大量使用  checkout 造成容易出现问题

基础学习建议看：<https://www.liaoxuefeng.com/wiki/896043488029600/898732864121440>  但里面关于 有些是过时的。请参考本文档使用

### 问题

如果发现本文档描述的，与实际使用不同的地方。请告知，我做相应处理。

### 持续发布

![](img/gitflow.png)

#### 核心点

* master 为核心开发分支

* master 是核心受保护分支

* 一切开发围绕master
  
  1. master 最新（除本地）
  2. 开发者不能直接操作 master 、pre-production 、 production
  3. 新的开始均从 master 拉取自己的本地分支，一般  feature-xxx 、 fix-xxx 、 开发自己创建并推送远程
  4. 开发人员测试后，提交合并请求
  5. 审查后合并到 master
  6. 预发布从 master 合并到 pre-production
  7. 发布从   pre-production 合并到 production ，打上 tag ，完成上线

* 必须遵循 fix 、feature ——> master ——> pre-production ——> production （视情况 pre 可略）

* 适当情况可取消 pre-production 

* production  发布必须打 tag

#### 分支管理

除 master 、pre-production 、production  为固定受保护分支，其他开发分支为临时分支，开发完成合并后可能会被删除

* 临时分支
  1. 功能分支 `feature` - 用于新功能的开发，建议以`issue-feature-name`命名
  2. 修复分支`fix` - 用户bug的修复，建议以`issue-fix-name`命名
* 固定分支
  1. 主分支 `master` - 用于发布到测试环境，上游分支为 `feature` 和 `fix`，该分支为**受保护分支**
  2. 发布分支 `stable` - 用于发布到预发环境，上游分支为 `master`，建议以`version-stable`命名，该分支要尽可能晚的创建，每次更新此分支都要更新一个小版本号

### 核心概念

#### 三大分区区

![gitstep.jpg](./img/gitstep.jpg)

* Working Tree 当前的工作区域

* Index/Stage 暂存区域，和git stash命令暂存的地方不一样。使用 `git add` xx，就可以将xx添加近Stage里面，真正保存文件

* Repository 提交的历史，即使用 `git commit` 提交后的结果，引用地址
  
  另外还有 Remote 远程仓库

#### 四大状态

三分区对应四大状态

* untrack   新建；  `git add`命令来track它

* modified  修改或者新增；通过 `git status` 查看，new file 新增，modified 修改

* staged  暂存； 通过 `git add `后的文件

* committed 提交； `git commit` 提交本地
  
  另外 git push 提交到远程仓库

#### 重要文件

* config 文件： 就是一些配置

* HEAD 文件：  指向当前正在工作的分支，此时任何 commit，默认自动附加到 指定分支之上

* logs/ 文件夹： 记录了每次的日志，`git reflog` 需要

* hooks/ 文件夹：用于在 git 命令前后做检查或做些自定义动作
  
  ```textile
  prepare-commit-msg.sample  # git commit 之前，编辑器启动之前触发，传入 COMMIT_FILE，COMMIT_SOURCE，SHA1
  commit-msg.sample          # git commit 之前，编辑器退出后触发，传入 COMMIT_EDITMSG 文件名
  pre-commit.sample          # git commit 之前，commit-msg 通过后触发，譬如校验文件名是否含中文
  pre-push.sample            # git push 之前触发
  
  pre-receive.sample         # git push 之后，服务端更新 ref 前触发
  update.sample              # git push 之后，服务端更新每一个 ref 时触发，用于针对每个 ref 作校验等
  post-update.sample         # git push 之后，服务端更新 ref 后触发
  
  pre-rebase.sample          # git rebase 之前触发，传入 rebase 分支作参数
  applypatch-msg.sample      # 用于 git am 命令提交信息校验
  pre-applypatch.sample      # 用于 git am 命令执行前动作
  fsmonitor-watchman.sample  # 配合 core.fsmonitor 设置来更好监测文件变化
  ```

* objects/ 文件夹:  Git的数据库
  
  1. pack 文件夹：  里面有  .pack 存储的自定义 `packfile` 的二进制格式，zlib压缩的文件数据   .idx 是索引文件。
  
  2. 很多两位字符的文件：对象的SHA1哈希值的前两位是文件夹名称，后38位作为对象文件名

* COMMIT_EDITMSG 文件：  最后一次提交的  message

* index 文件， 暂存文件存放，是一个二进制文件

* fefs/ 文件夹 ： 一般包括三个子文件夹，`heads`、`remotes`和 `tags `
  
  1. `refs/heads/` 文件夹内的 `ref` 一般通过 `git branch` 生成。`git show-ref --heads` 可以查看。
  
  2. `refs/tags/` 文件夹内的 `ref` 一般通过 `git tag` 生成。`git show-ref --tags` 可以查看。

#### 分支

* 远程分支
  
  在远程仓库中的分支，就是像本地分支一样的普通分支，只不过是在远程仓库里

* 远程跟踪分支
  
  本地仓库中用来代表远程分支的本地分支，它是只读的，只是表示远程分支的状态，本地不可修改

* 跟踪分支
  
  已跟踪了远程跟踪分支的本地分支，可以通过pull和push命令快捷地获取合并数据和推送数据

远程跟踪分支 是 跟踪分支 与 远程分支 之间的跟踪桥梁

### 核心操作

#### config 配置

1. 设置用户名和邮件地址
   
   ```shell
   git config user.name 'xxxx'
   git config user.email 'xx@x.x'
   
   # 可带参数 
   --global              use global config file
   --system              use system config file
   --local               use repository config file
   --worktree            use per-worktree config file
   优先级逐渐变高
   
   git config --local -l 查看仓库配置
   ```

2. 使用http连接，需要每次输入密码
   
   ```shell
   git config --global credential.helper store --file ~/.my-credentials
   # --file ~/.my-credentials 表示存储的地址 ， 可以不加，会自动存储在这个默认的目录、文件
   ```

#### branch 本地分支

1. 检出
   
   **不推荐使用checkout，因为 checkout 是危险的**
   
   ```shell
   git checkout https://github.com/xiaocanyun/rebase.git
   # 完整的检出目标仓库
   
   # 可使用如下代替
   mkdif xxx # xxx 是目录
   git init # 初始化
   git remote add origin https://github.com/xiaocanyun/pangu.git # 添加某远程仓库
   可使用 git push/pull/fetch  origin master 推送\拉取
   ```
   
   **最多在检出使用 checkout 任何时候都不要再使用**

2. 创建分支
   
   ```shell
   git branch xxx
   # 不建议使用  git checkout -b $issue-feature-name  创建并切换 （该命令以后可能不能使用）
   # 从当前分支创建一个新分支，不会切换过期
   ```

3. 切换
   
   ```shell
   git switch xxx 
   # 从当前分支切换到 xxx 分支
   ```

4. 推送
   
   ```shell
   git push origin dev
   # 推送到远程 分支
   # origin 是默认的一个当前环境的 remote 地址的 别名，一般不用改
   # dev 是远程分支的名称
   ```

5. 删除
   
   ```shell
   git branch -d  ${branch-name}
   # 删除本地分支
   ```

#### commit 提交

提交只会对本地当前分支产生作用

1. 直接提交
   
   ```shell
   git commit  -m "你的注释"
   ```

2. 修改最后一次提交
   
   ```shell
   git 
   ```

#### commit 规范

commit 遵循业界优秀的范例 Angular 项目团队所使用的规范，进行简单的约定

[Commits · angular/angular · GitHub](https://github.com/angular/angular/commits/main)

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

```
fix(core): improve TestBed declarations standalone error message (#45999)

improve the error message developers get when adding a standalone
component in the TestBed.configureTestingModule's declarations array,
by making more clear the fact that this error originated from the
TestBed call

resolves #45923

PR Close #45999
```





##### 基本约束

* 每行不能超过80个字符，避免换行影响阅读

* 适用换行，必须空一行空行

##### header

只有一行

```
<type>(<scope>): <subject>
```

```
fix(): 修改新建用户注册的短信验证码长度为6
```

1. type 必须，表明类型，如下可选
   
   * feat： 新功能
   
   * fix：修补bug
   
   * perf：性能优化（未新增功能、未修改bug、未修改业务）
   
   * docs：文档
   
   * style：格式（只是格式变更）
   
   * refactor：重构（组件升级、优化等，未新增功能、未修改bug、未修改业务）
   
   * test：测试
   
   * chore  构建或者辅助工具变动
   
   如 type为 feat 或 fix ，则一定会出现在 git log 的 change log 中，其他的可以设置为不显示

2. scope 必须，表明范围，如下可选
   
   * rest： web接口变动
   
   * service： 服务层变动
   
   * interface：程序接口变动
   
   * database：数据库变动
   
   * utils：工具的变更
   
   * depend：依赖变更
   
   * persistent： 持久层变动
   
   如果涉及到多个，请使用半角逗号分隔

3. subject 必须，对本地 commit 的简短描述，
   
   * 不超过 50 字符 
   
   * 除通用名词，使用中文
   
   * {变更方式}{变更模块}{变更内容}
   
   * 除专用名词，均小写
   
   * 不出现标点符号
   
   类似： 增加\修改\删除\新建用户注册的短信验证码长度

##### body

 对每次 commit 的详细描述，

* 每行不超过 80 字符

* 可分隔多行

* 除通用名词，使用中文

* 除专用名词，均小写

* 少用标点符号

类似:

```
如有必要，请提供更详细的解释文本。将其包装到大约72个字符左右。

如果换行，必须空一行空行

- 列出几个重要点
- 另一个
```

##### footer   当前定义在该情况下使用

*  引用关闭某个 issue 或者 某个具体的问题id
  
  ```
  Closes #234 #222
  ```
  
  

#### remote 远程分支

远程分支

1. 查看当前远程仓库
   
   ```shell
   git remote  -v  # 查看关联的所有的远程仓储名称及地址
   git remote  #查看所有的远程仓储名称
   ```

2. 添加远程分支
   
   本地建立一个远程分支的记录 与分支没关系 
   
   如果在 git push 使用 -u 则就默认给当前分支绑定一个远程分支，以后就可以不输入远程名称
   
   **不建议使用 -u 自己指定远程更清晰**
   
   ```shell
   git remote add  {branch_name} https://github.com/xxxs.git
   # 创建跟踪关系
   git push origin {branch_name} 
   # 即可提交，如果服务器未创建 {branch_name} 分支，会自动创建，并提交文件
   # 注意会将 该分支 的全部 log 一并 push 到新建分支 也就是说新建分支有原分支的所有历史
   # 这个可以通过  TODO 来避免
   ```

3. -u 详解 **谨慎使用**
   
   如果使用了 -u 会有如下效果，自动绑定一个，如果没有就不会默认绑定一个，每次fetch、push需要输入
   
   根目录的 .git/ 目录下，config
   
   ```
   [branch "master"]
       remote = origin
       merge = refs/heads/master
   [branch "test"]
       remote = origin
       merge = refs/heads/test
   ```

4. 删除远程仓库
   
   **谨慎操作**
   
   ```shell
   git remote remove ${branch-name}
   ```

#### diff 比较

1. 比较工作区与暂存区
   
   ```shell
   git diff           # 查看工作区和暂存区的区别； 后面可跟文件
   ```

2. 比较暂存区与本地仓库(已 commint)
   
   ```shell
   git diff --cached  # 查看暂存区和上次提交的区别； 后面可跟文件  
   ```

#### rm 删除

1. 删除工作区和暂存区
   
   ```shell
   git rm 文件     # 删除暂存区、和当前工作区的某个文件
   ```

2. 只删除暂存区
   
   ```shell
   git rm --cached 文件   #  只删除暂存区的文件
   ```

3. 删除本地仓库
   
   见  rest

#### statu 状态

1. 查看当前工作区状态
   
   ```shell
   git status  # 查看当前工作区与暂存区的差异
   ```

#### log 日志

注意事项 **核心是指针**

* 比如 master 提交了很多，然后从 master 创建 branch ，branch 查看log  会有master的历史

* 如果将 branch ，merge 到 master 那么 master 也会有 branch 的 commit log 

* 针对上一条，如果不想要 branch 的明细  commit log。 那么合并到 master 使用 rebase 
1. 查看当前分支 commit 日志
   
   ```shell
   git log  # 查看当前分支的 commit 的历史日志
   ```

2. 详细日志，时间轴, 以 当前分支 为主线线上
   
   **注意显示的是 commit log ， 因为合并产生的 log 也在其中 **
   
   ```shell
   git log --graph --decorate --oneline --all
   # --decorate： 标记会让git log显示每个commit的引用
   # --oneline： 一行显示
   # --graph   点线图
   # --simplify-by-decoration：只显示被branch或tag引用的commit
   # --all: 所有分支
   ```
   
   ```
   *表示一个commit， 注意不要管*在哪一条主线上
   |表示分支前进
   /表示分叉
   \表示合入
   ```

3. 查看操作日志
   
   ```shell
   git reflog  # 查看当前分支 的 操作历史
   ```

#### fetch 本地更新

不赞成使用  pull

1. 拉取远程 fetch
   
   ```shell
   git fetch origin master # 从 远程 origin，分支 master 拉取最新到远程跟踪分支
   ```

2. 查看差异
   
   ```shell
   git log -p dev.. origin/dev 
   # 查看本地 dev  和 远程跟踪仓库 orgin/dev 的 commit 差异
   ```

3. 合并 
   
   ```shell
   git merge origin/dev
   # 把 origin/master 分支合并到当前分支
   # 前提是先 fetch
   
   # 
   ```

#### restore  工作区、暂存区

1. 将工作空间恢复到暂存状态
   
   ```shell
   git restore  文件   #  指定路径或者文件
   # 只有已经 add 到暂存的才会
   # 相当于 本地 ctrl + z
   ```

2. 将暂存区 恢复到 最后的提交状态，工作区不受影响
   
   ```shell
   git restore --staged 文件  # 指定路径或者文件
   # 相当于取消之前 add 到暂存区的， 本地文件不会受影响
   # 如果一个文件已经 多次 add 到 暂存区，均会撤销  
   # 与 git add 相反
   # 不推荐使用旧版的   git reset HEAD <file>  
   ```

3. 将本地未提交的清除，即将工作区和暂存区都变更为指定的提交版本
   
   **基本上push都会冲突**
   
   ```shell
   git reset --hard HEAD # HEAD 可指定 commit id
   # --hard还告诉Git也一并覆盖工作区的所有改动
   # 工作区、暂存区 变更为 指定的版本
   # push生效
   ```

#### reset 重置 - 版本库（不使用）

reset 会消除之前的log 和 reflog ，是永恒的撤销

1. --soft  将 HEAD(版本库) 指向指定的 commit
   
   ```shell
   git reset --soft HEAD^  # 可以是 commit id
   # 工作区 和 暂存区 不变
   # git log 查看，指定的 commit id 后面的 commit 丢失
   # 如果马上提交 git commit -m '' 将最新的暂存提交。达到的效果相当于删除commit id后的提交记录，把最新的新增一个记录(就是之前没操作 rest 时的最新记录)
   # 这里的马上，是指 在 reset 后没有再执行 git add ，因为 add 了会改变暂存区
   ```

2. --mixed(默认) 将版本库（HEAD）和暂存区，不会修改工作区
   
   ```shell
   git reset --mixed HEAD^ # 可是 commit id
   # git log 查看，指定的 commit id 后面的 commit 丢失
   # 暂存区已改变为与版本库一样
   ```

3. --hard 将本地未提交的清除，即将工作区和暂存区都变更为指定的提交版本
   
   **可能会出现冲突**
   
   ```shell
   git reset --hard HEAD # HEAD 可指定 commit id
   # --hard还告诉Git也一并覆盖工作区的所有改动
   # 工作区、暂存区 变更为 指定的版本
   # push生效
   ```

#### revert 重写 - 版本库

 其作用是创建新的提交来弥补之前提交的错误

1. 撤销工作区到指定的版本
   
   **可能会出现冲突**
   
   ```shell
   git revert HEAD # HEAD 可使用 HEAD^ 或者使用 commint id 来指定
   # 会将工作区内容直接返回到 指定的 commit id 处，即变更了工作区,可能会和本地冲突
   # 并创建新的提交（有些版本，不会自动提交，只是放到 自动暂存区需要重新提交）
   # push 到远程
   # 其导致的结果是：1. 工作区恢复、2. commit 的 log 增加一个 3. 之前的任何操作记录不会变化
   ```
   
   然后处理冲突，并提交，即增加新的提交来弥补之前错误的提交

2. 强制结束 revert 产生的影响 --abort
   
   如果使用了 git revert 来操作，但是不想继续。可使用 -- abort 来强制结束
   
   ```shell
   git revert --abort
   ```

3. 按顺序一步一步回滚，然后一起处理
   
   如果提交了 1、2、3、4  但是其中第 2 处出现了问题，想修复
   
   这样如果本地都是提交了的，那么顺序撤销，不会产生冲突
   
   因为如果直接 git revert 2 。 很可能会冲突
   
   ```shell
   git revert 4 --no-commit
   git revert 3 --no-commit
   git revert 2 --no-commit
   git revert --continue
   # 此时提交 ，即变成 2 的时候的
   ```

#### stash 临时存储

需要临时 switch 到其他分支工作，但当前分支还不可用提交

1. stash list 查看当前存储列表(强烈建议每次只使用一次 存储，即 list 只有一条)
   
   ```shell
   git stash list 
   # 查看 stash 列表  -p 可查看详细
   
   git show stash@{id} 
   # 查看制定存储与当前工作区差异  -p  可查看详细
   ```

2. 存储当前分支环境
   
   注意，必须当前工作区文件均要被 git 管理( 当然除了 .gitignore  )
   
   ```shell
   git stash
   # 存储当前环境
   
   git stash save 'xawewe' 
   # 可以命个名
   
   git status
   # 再次查看工作区已经 是干净的了 
   # 现在可创建分支、或者切换分支进行操作
   ```

3. 还原临时存储
   
   需要先切换到之前的分支
   
   ```shell
   git switch xxx 
   # 切换到之前工作的分支(使用了临时存储的分支)
   
   git stash list 
   # 查看当前临时存储列表
   # -p 可以查看详细
   
   git stash apply # 恢复之前的存储，但是不删除
   git stash drop  # 删除
   # 上面两个可以带 stash 的标识，通过list 可查看(但是可能会冲突，故最好指暂存一次)
   
   git stash pop  # 恢复并删除最新的一个（栈结构）
   ```

4. 清除
   
   **谨慎操作**
   
   ```shell
   git stash clear
   # 清除当前所在分支本地创建的 stash
   ```

#### 合并 rebase  & merge

需要细致理解 rebase 与 merge 的差异

* 本地开发分支合并父分支 使用 merge
  
  **对于分支开发者，可一直使用 merge**

* 合并开发分支到父分支使用 rebase  变基
  
  **受保护分支合并，使用 rebase**
  
  rebase 的含义就是以 指定的 branch 为基，来进行合并，会重置commit的时间线
  
  也就是说，不按照正常的提交时间点进行合并，把所指分支当作当前分子基线。

#### rebase 变基合并

1. rebase 变更分支的基、重树分支 commit log
   
   **一般情况不需要使用，强制工作中不使用**
   
   ```shell
   比如 有 master 其  commit id
   -- m1、m2、m3、m4、m5
   branch 在 m3 处创建分支 a  然后自主演进 commit id
   -----------\、 a1、a2
   branch 在 a1 处创建分支 b  然后自主演进 commit id
   ---------------\、b1、b2
   
   git switch b
   # 进入 b 分支
   git rebase master 
   会变成如下
   -- m1、m2、m3、m4、m5
   -------------------\、b1、b2
   -----------\、 a1、a2
   
   1. 以 master 为基
   2. 从 master 最新开始
   3. 然后重树 b 后面的 commit id
   4. 可能会产生冲突
   ```

2. rebase 有很多参数， -i 进入交互式环境可以操作
   
   在本分支。提交了很多代码，然后需要合并其他分支的代码的时候。
   
   可以再次对本次提交的代码进行 多次提交的合并（这个用于当需要将本地开发分支提交到 开发主分支 dev）
   
   ```shell
   git rebase -i 你需要合并的基分支
   # 进入交互环境 会出现编辑框。
   # 默认是 pick
   # 将某些项改为 s  。 即代表合并
   # 保存即可，根据提示操作
   ```

#### merge 合并

1. 合并
   
   **主意处理冲突**
   
   ```shell
   git merge  xxx
   # 合并 xx 分支 到本分支
   # 会完整保留两者各自的分支  commit log
   ```

2. 多commit 压缩合并
   
   **主意处理冲突**
   
   ```shell
   git merge xxx --squash
   # 使用squash方式合并，把分支多次commit历史压缩为一次 
   # 会把 xxx 分支的与当前分支差异加入当前分支工作区、和暂存区
   # 但是忽略 xxx 分支的 commit log
   git commit -m ""
   # 重新在本分支执行一次提交即可
   # 因为将合并过来的只加入了工作区和暂存区
   # 可以通过   git restore --staged .   及     git restore . 取消 
   ```

#### cherry-pick  略，不建议使用

### 版本发布

在于同时维护多个版本，该模型在当前环境不太适用。其核心点在于

* 每次发布均从测试良好后的 master 拉一个分支 1-2-stable 
* 该稳定发布版本只接收 bug 修复 
* 每次修复后需打上 一个 更小的 tag 
* 发布

其他不再详述。

### tag

1. 打tag
   
   ```shell
   git tag -a v0.1 -m "version 0.1 released" 1094adb
   # -a 指定标签名  可略
   # -m 指定说明文字  可略
   # 1094adb 从哪个 commitid 打tag  如果略了，就是当前最新 commit
   # 可使用 git log --oneline 查看历史 commit
   ```

2. 查看tag
   
   ```shell
   git tag  # 查看所有标签
   git show v0.1  # 查看 标签信息
   ```
